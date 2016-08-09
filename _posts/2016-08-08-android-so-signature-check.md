---
layout:     post
title:      "Android so库防客户端破解的解决方案"
subtitle:   " \"Keep fighting!\" "
date:       2016-08-08 12:00:00
author:     "leehong"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Android
    - 安全性
---


随着移动互联网的发展，移动应用的安全问题越来越突显，特别是涉及到钱相关的产品，前段一段时间，我们的Android客户端产品被人破解了，修改了一些代码，重新打包签名后，就可以免费获得资源，对我们的收入造成了一些影响，移动产品安全性就不得不提重视，安全性这个话题很大，包括客户端、服务端、数据存储、协议等很多方面，这里只是从客户端的角度来讨论一上如何保证客户端产品的安全性，抛砖引玉，也希望大家多提意见和建议。


下面主要从以下几个方面来展开讨论：

---

## C/S协议安全

在不使用https的前提下，要保证C/S协议的安全，一般都会进行参数的校验，以及参数加密，客户端和服务端会约定一个固定的字符串作为key，对于客户端来说，这个key应该放到哪里？最早之前，我们是直接放到Java代码中，这样可以说没有什么安全性，后来为了稍微更加安全一点，把这些key都统一放到so库中实现，虽然也不能保证绝对安全，但起码可以增加破解的难度。

你肯定要说，如果这样做，就会有一个问题，如果别人把so拿出来，在直接调用这些native接口，也同样可以获得key，所以也同样不安全，怎么办呢？ 


能不能让 so 库只能在我们自己的app运行，别人调用就是砖头呢？

各位看官，接着往下看。

## so库校验签名

一般的情况下，都会在 `Application.onCreate()` 方法里面检查当前应用的签名是否合法，如果不合法就直接退出，这种情况其实无法正在防止破解，因为破解的可以找到调用入口，把相应的代码删除，所以这样方法也就失效了，那有没有更好的方案呢？

想到的一种思路就是，so库本身就具体签名校验的机制，当so库被加载时 （`JNI_OnLoad()`方法），如果签名不合法，直接失败，so库根本加载不起来。

大概的思路如下代码所示：

```cpp
jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env;
    if (vm->GetEnv((void **) (&env), JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }

    LOGI("Library JNI_OnLoad begin =========");

  if (checkSignature(env) != JNI_TRUE) {
            LOGE("    The app signature is NOT correct, please check the apk signture. ");
            LOGI("Library JNI_OnLoad end ===========");
            return -1;
    } else {
            LOGI("    The app signature is correct.");
    }

    LOGI("Library JNI_OnLoad end ===========");

    return JNI_VERSION_1_6;
}
```

__说明：这里有一个问题，需要注意，在开发过程中，我们不需要检查签名的合法性，只有在release版本才检查，所以上述逻辑还再完善，需要添加上DEBUG和RELASE的判断。__

__要怎么判断呢？我目前的思路是通过宏来判断，如果定义了宏并且为`JNI_TRUE`的话，就认为是release版本。__

以下是完整的实现：

```cpp
jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env;
    if (vm->GetEnv((void **) (&env), JNI_VERSION_1_6) != JNI_OK) {
        return -1;
    }

    LOGI("Library JNI_OnLoad begin =========");

    // RELEASE_MODE这个宏是通过编译脚本设定的，如果是release模式，
    // 则RELEASE_MODE=1，否则为0或者未定义
#ifdef RELEASE_MODE
    if (RELEASE_MODE == 1) {
        // 检查当前应用的签名是否一致，如果不签名不一致的话，则直接退出
        if (checkSignature(env) != JNI_TRUE) {
            LOGE("    The app signature is NOT correct, please check the apk signture. ");
            LOGI("Library JNI_OnLoad end ===========");
            return -1;
        } else {
            LOGI("    The app signature is correct.");
        }
    } else {
        // Do nothing
    }
#endif

    LOGI("Library JNI_OnLoad end ===========");

    return JNI_VERSION_1_6;
}
```

#### `RELEASE_MODE` 在哪里定义的？

那问题又来了？ `RELEASE_MODE` 宏要在哪里定义？不能改代码吧？

很容易想到通过编译中的`buildTypes`来控制，如果当前打release包，那么就定义这个宏。请看Android Stuido的build.gradle中的buildTypes

```java
buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

            ndk {
                // release包定义RELEASE_MODE=1宏，so库中会使用
                cFlags "-DRELEASE_MODE=1"
            }
        }
        debug {
               // do nothing
        }
    }
```

我们在buildTypes中添加了一个ndk块，里面定义了 `RELEASE_MODE` 宏，这里使用了 cFlags。

__注意：`-DRELEASE_MODE=1` 中前面的 -D 不能省略。__

#### checkSignature()如何实现？

__最主要的就是解决如何得到当前app的context，我的做法是反射Java层的一个特定的方法__，返回 `getAppContext()` 方法得到context。其中 `setAppContext()` 方法在 Application.onCreate()中调用。 

```java
public class NativeContext implements NoProGuard {
    /**
     * App context
     */
    private static Context sAppContext;

    /**
     * 得到 app context
     */
    public static Context getAppContext() {
        return sAppContext;
    }

    /**
     * Set the app context
     */
    static void setAppContext(Context appContext) {
        sAppContext = appContext;
    }
}
```

Native这一层的实现如下：

```cpp
/**
 * 检查加载该so的应用的签名，与预置的签名是否一致
 */
static jboolean checkSignature(JNIEnv *env) {
    // 得到当前app的NativeContext类
    jclass classNativeContext = env->FindClass(CLASS_NAME_NATIVECONTEXT);
    // 得到getAppContext静态方法
    jmethodID midGetAppContext = env->GetStaticMethodID(classNativeContext,
                                                        METHOD_NAME_GETAPPCONTEXT,
                                                        METHOD_SIGNATURE_GETAPPCONTEXT);
    // 调用getAppContext方法得到context对象
    jobject appContext = env->CallStaticObjectMethod(classNativeContext, midGetAppContext);

    if (appContext != NULL) {
        jboolean signatureValid = Java_com_xxxx_android_AppRuntime_checkSignature(env, NULL, appContext);
        if (signatureValid == JNI_TRUE) {
            LOGI("    checkSignature() return true");
        } else {
            LOGI("    checkSignature() return false");
        }
        return signatureValid;
    }

    return JNI_FALSE;
}
```

这里调用了 `Java_com_xxxx_android_AppRuntime_checkSignature` 方法，它的实现如下所示，核心的思路是将从 `Context` 里面得到当前app的签名MD5字符串，然后再与预置的常量作比较，调用了 `strcmp` C函数。

```java
extern "C" JNIEXPORT jboolean JNICALL
Java_com_xxxx_android_AppRuntime_checkSignature(
        JNIEnv *env, jclass clazz, jobject context) {

    jstring appSignature = loadSignature(env, context);
    jstring releaseSignature = env->NewStringUTF(APP_SIGNATURE);
    const char *charAppSignature = env->GetStringUTFChars(appSignature, NULL);
    const char *charReleaseSignature = env->GetStringUTFChars(releaseSignature, NULL);

    jboolean result = JNI_FALSE;
    if (charAppSignature != NULL && charReleaseSignature != NULL) {
        if (strcmp(charAppSignature, charReleaseSignature) == 0) {
            result = JNI_TRUE;
        }
    }

    env->ReleaseStringUTFChars(appSignature, charAppSignature);
    env->ReleaseStringUTFChars(releaseSignature, charReleaseSignature);

    return result;
}
```

__APP_SIGNATURE__ 是const char*的常量，是release版本的签名字符串。

完整的代码如下：

```cpp
jstring ToMd5(JNIEnv *env, jbyteArray source) {
    // MessageDigest类
    jclass classMessageDigest = env->FindClass("java/security/MessageDigest");
    // MessageDigest.getInstance()静态方法
    jmethodID midGetInstance = env->GetStaticMethodID(classMessageDigest, "getInstance", "(Ljava/lang/String;)Ljava/security/MessageDigest;");
    // MessageDigest object
    jobject objMessageDigest = env->CallStaticObjectMethod(classMessageDigest, midGetInstance, env->NewStringUTF("md5"));

    // update方法，这个函数的返回值是void，写V
    jmethodID midUpdate = env->GetMethodID(classMessageDigest, "update", "([B)V");
    env->CallVoidMethod(objMessageDigest, midUpdate, source);

    // digest方法
    jmethodID midDigest = env->GetMethodID(classMessageDigest, "digest", "()[B");
    jbyteArray objArraySign = (jbyteArray) env->CallObjectMethod(objMessageDigest, midDigest);

    jsize intArrayLength = env->GetArrayLength(objArraySign);
    jbyte* byte_array_elements = env->GetByteArrayElements(objArraySign, NULL);
    size_t length = (size_t) intArrayLength * 2 + 1;
    char* char_result = (char*) malloc(length);
    memset(char_result, 0, length);

    // 将byte数组转换成16进制字符串，发现这里不用强转，jbyte和unsigned char应该字节数是一样的
    ByteToHexStr((const char*)byte_array_elements, char_result, intArrayLength);
    // 在末尾补\0
    *(char_result + intArrayLength * 2) = '\0';

    jstring stringResult = env->NewStringUTF(char_result);
    // release
    env->ReleaseByteArrayElements(objArraySign, byte_array_elements, JNI_ABORT);
    // 释放指针使用free
    free(char_result);

    return stringResult;
}

jstring loadSignature(JNIEnv *env, jobject context) {
    // 获得Context类
    jclass cls = env->GetObjectClass(context);
    // 得到getPackageManager方法的ID
    jmethodID mid = env->GetMethodID(cls, "getPackageManager", "()Landroid/content/pm/PackageManager;");

    // 获得应用包的管理器
    jobject pm = env->CallObjectMethod(context, mid);

    // 得到getPackageName方法的ID
    mid = env->GetMethodID(cls, "getPackageName", "()Ljava/lang/String;");
    // 获得当前应用包名
    jstring packageName = (jstring) env->CallObjectMethod(context, mid);

    // 获得PackageManager类
    cls = env->GetObjectClass(pm);
    // 得到getPackageInfo方法的ID
    mid = env->GetMethodID(cls, "getPackageInfo", "(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");
    // 获得应用包的信息
    jobject packageInfo = env->CallObjectMethod(pm, mid, packageName, 0x40); //GET_SIGNATURES = 64;
    // 获得PackageInfo 类
    cls = env->GetObjectClass(packageInfo);
    // 获得签名数组属性的ID
    jfieldID fid = env->GetFieldID(cls, "signatures", "[Landroid/content/pm/Signature;");
    // 得到签名数组
    jobjectArray signatures = (jobjectArray) env->GetObjectField(packageInfo, fid);
    // 得到签名
    jobject signature = env->GetObjectArrayElement(signatures, 0);

    // 获得Signature类
    cls = env->GetObjectClass(signature);
    // 得到toCharsString方法的ID
    mid = env->GetMethodID(cls, "toByteArray", "()[B");
    // 返回当前应用签名信息
    jbyteArray signatureByteArray = (jbyteArray) env->CallObjectMethod(signature, mid);

    return ToMd5(env, signatureByteArray);
}

void ByteToHexStr(const char *source, char *dest, int sourceLen) {
    short i;
    char highByte, lowByte;

    for (i = 0; i < sourceLen; i++) {
        highByte = source[i] >> 4;
        lowByte = source[i] & 0x0f;
        highByte += 0x30;

        if (highByte > 0x39) {
            dest[i * 2] = highByte + 0x07;
        } else {
            dest[i * 2] = highByte;
        }

        lowByte += 0x30;
        if (lowByte > 0x39) {
            dest[i * 2 + 1] = lowByte + 0x07;
        } else {
            dest[i * 2 + 1] = lowByte;
        }
    }
}
```

以下就是在native层实现签名检验的逻辑，下面接下来说一下如何编译。

### 如何编译，mk vs gradle

创建jni的过程，这里就不多说，网上有很多相关的文章，简单地说，就是 `src/main/` 路径下，创建jni文件夹，然后把native的代码放在这个目录下，在根目录下在，创建对应的Android.mk和Application.mk文件，关于 `Android.mk` 和 `Application.mk` 的说明，请参考Android开发的官方文档：

* [Android.mk](https://developer.android.com/ndk/guides/android_mk.html)
* [Application.mk](https://developer.android.com/ndk/guides/application_mk.html)

如果是使用gradle构建的话，需要作一点配置，添加一个 `ndk` block，gradle里面的配置会覆盖 mk 中设置的。

```java
defaultConfig {
        ...

        ndk {
            // so库的名字
            moduleName 'libAppRuntime_V1_0'
            // 支持armeabi和armeabi-v7a
            abiFilters("armeabi", "armeabi-v7a")
            // 依赖的类库
            ldLibs("log")
        }
    }
```

 * moduleName：so库的名字
 * abiFilters：so库的平台
 * ldLibs：依赖的类库，这里需要输出Log到logcat中，所以依赖log这个Android提供的库


### Android Gradle插件如何编译出library module的debug包

__为什么需要debug模式的library呢？因为我是想主工程打debug模式的包，library也是走debug的配置，如果主工程是release配置模式，那么library也是release的配置，这样做的目的是为了定义 RELEASE_MODE=1 这样的宏，是为了控制在不同的版本的so库执行不同的业务逻辑。__

由于上述的功能逻辑是放到一个library module中的，那么就面临一个问题，library module如何打出debug包，对于library Android默认是打出release模式的，这一点可以从编译出来的`BuildConfog.DEBUG` 恒为false可以看出。

那么到底要如何才能打出debug的aar呢？为了实现这个功能，再真费了点劲。直接上结论！

参考文档：[Gradle插件不能编译出library模块的DEBUG模式](http://stackoverflow.com/questions/28081846/use-different-build-types-of-library-module-in-android-app-module-in-android-stu)

文中也有人说到了不能打debug模式的包：
> Well, Gradle Android plugin simply can't build the debug version of dependent library modules. This is a well-known, old issue and this is not resolved yet.
You can try to use some workarounds from the discussion I mentioned, specifically take a look at posts #35 and #38.

解决方案也大概如文中所说的，也进行了多次尝试：

#### __主工程依赖方式需要改：__

通常是这样引用libraray module
> compile project(':AppLibrary')

需要改成这样：
> debugCompile project(path: ':AppLibrary', configuration: 'debug')
releaseCompile project(path: ':AppLibrary', configuration: 'release')

### __再看看library modlue的gradle配置：__

* 首先 `buildTypes` 增加 `debug` 和 `release` 配置块
* 在 `android` 块里面增加 `publishNonDefault true`

这样改后，我们编译后就可以发现生成的文件中会有debug和release两个文件夹了，如下图所示：

![library_module_build](/img/2016/2016-08-08-android-so-signature-check-01.png)

### JNI开发的一些tips


官方的tips请点击这里：[JNI Tips](https://developer.android.com/training/articles/perf-jni.html?hl=ko)

在开发过程中，遇到了一些比较蛋疼的问题，给大家说说，避免踩坑。  


#### 生成 .h 头文件
如果native方法中引用了android的类，例如Context之类的，需要显示指定--classpath
参考链接：[android - javah doesn't find my class](http://stackoverflow.com/questions/7635624/android-javah-doesnt-find-my-class)

If you are on Linux or MAC-OS, use ":" to separate the directories for classpath rather than ";" character: Example:

> javah -cp /Users/Android/android-sdk/platforms/android-xy/android.jar:. com.test.JniTest
<br>

#### JNI so库未找到方法实现

__如果实现是C++（后续是cpp），没有头文件(.h)的话__，需要在接口实现处添加上  `extern "C"`，简单地说，C++的实现需要向前兼容C的实现，关于 extern "C"的作用，这里不多讲，
可以参考：[extern "C"用法解析](http://www.jianshu.com/p/5d2eeeb93590)

```cpp
extern "C" JNIEXPORT jboolean JNICALL
Java_com_xxxx_android_AppRuntime_checkSignature(
```

### 总结

1、上述的东西，可以再进一步封装，独立成为一个libaray module，提供一个sdk，输出的就是aar，Java层面上就是一个NativeContext类，这个类的`setAppContext()`接口必须由业务方来调用。

2、上述提到的 so 最大的一个好处是自己具备识别签名的能力，只能在我们自己的app里面使用，别人是无法用的

3、由于ndk开发经验不多，这个东西花了我两天时间学习和研究，幸好之前做过两年多的C++开发，对C++这方面还比较熟，所以慢慢也上手了，更多是一个熟悉的过程。所以，总结到这里，以备随时翻阅~~

各位看官，有好的建议，欢迎留言，写了这么多，点个赞呗~~~
<br>
