---
layout:     post
title:      "Android setPolyToPoly遇到的问题"
subtitle:   " \"A burden of one’s choice is not felt.\" "
date:       2017-06-04 12:00:00
author:     "leehong"
header-img: "img/post-bg-android.jpg"
catalog: true
tags:
    - Android
---

### 问题

最近遇到一件比较诡异的问题，使用 `Matrix.setPolyToPoly` 接口绘制一个多边形，在 **`华为`** 手机上遇到变形的问题。

直接上图，看图说话：

![](/img/2017/2017-06-05-canvas-issue-on-huawei-phone-01.png)

正确的图形应该是图片全部显示在蓝色边框的梯形之内，但实际上不正确，我的实现代码版本如下：

**计算Matrix**

```java
private void calcMatrix() {
    final int width = getWidth() - 3;
    final int height = getHeight();
    final int val = 30;
    final int rightDx = mBookLineDrawable.getIntrinsicWidth();

    final float[] src = {
        0, 0,
        width, 0,
        width, height,
        0, height
    };

    final float[] dst = {
        val, val,
        width - rightDx, 0,
        width - rightDx, height,
        val, height - val
    };

    mPath.reset();
    mPath.moveTo(dst[0], dst[1]);
    mPath.lineTo(dst[2], dst[3]);
    mPath.lineTo(dst[4], dst[5]);
    mPath.lineTo(dst[6], dst[7]);
    mPath.lineTo(dst[0], dst[1]);

    mMatrix.reset();
    mMatrix.setPolyToPoly(src, 0, dst, 0, src.length >> 1);
getHeight());
}
```

**绘制逻辑**

```java
@Override
protected void onDraw(Canvas canvas) {
    canvas.save();
    canvas.concat(mMatrix);
    super.onDraw(canvas);
    canvas.restore();
}
```

把 Matrix 作用到系统传递过来的 canvas 上面，理论上来说这应该是没有问题的，但不知华为系统对 canvas 作了什么处理，就是不正常。

### 解决办法

#### 方案一：关闭硬件加速？
怎么去解决这个问题呢？没着，一个一个地试吧，最后分析，一般跟绘制相关，硬件加速是一个比较重要的因素，所以尝试把硬件加载关闭，嘿，发现居然好了，但是又有一新问题，看图说话：

![](/img/2017/2017-06-05-canvas-issue-on-huawei-phone-02.png)

图形显示对了，但是周围有黑边，怀疑是 matrix 对 canvas 作用后，透明处理出了问题。这种方案不可行。

#### 方案二：自定义canvas？

再接着试吧，matrix 作用在系统的 canvas 上面有问题，那我们自己创建一个 canvas，先把图形画到自己的画布上，最后再画到系统 canvas 上，这样会不会有问题呢？抱着试一试的想法，去尝试一下，还居然好了。

正确的效果图如下：

![](/img/2017/2017-06-05-canvas-issue-on-huawei-phone-03.png)

**简单来说一下，就是先绘制到自己的 canvas(其实也就是bitmap) 上，再次这个 bitmap 绘制到系统的 canvas 上面。**

我封装了一个 CanvasWrapper 来实现这个功能。

**CanvasWrapper**

```java
class CanvasWrapper {

    private int mWidth;
    private int mHeight;
    private final Canvas mCanvas = new Canvas();
    private final Rect mDstRect = new Rect();
    private Bitmap mDrawBitmap;
    private final Paint mPaint = new Paint();

    public CanvasWrapper() {
        mPaint.setAntiAlias(true);
    }

    public void setSize(int width, int height) {
        if (width <= 0 || height <= 0) {
            return;
        }

        mWidth = width;
        mHeight = height;
        mDstRect.set(0, 0, width, height);

        createDrawBitmapIfNeed();
    }

    public void save() {
        reset();
        mCanvas.save();
    }

    public void restore() {
        mCanvas.restore();
    }

    public void concat(Matrix matrix) {
        mCanvas.concat(matrix);
    }

    public Canvas getCanvas() {
        return mCanvas;
    }

    public void drawBitmap(Bitmap bitmap) {
        if (null != bitmap) {
            mCanvas.drawBitmap(bitmap, null, mDstRect, mPaint);
        }
    }

    public void drawDrawable(Drawable drawable) {
        if (drawable != null) {
            drawable.draw(mCanvas);
        }
    }

    public void draw(Canvas canvas) {
        if (mDrawBitmap != null) {
            canvas.drawBitmap(mDrawBitmap, null, mDstRect, null);
        }
    }

    public boolean isValid() {
        return mDrawBitmap != null;
    }

    private void createDrawBitmapIfNeed() {
        final int width = mWidth;
        final int height = mHeight;
        if (null != mDrawBitmap) {
            if (mDrawBitmap.getWidth() != width || mDrawBitmap.getHeight() != height) {
                mDrawBitmap = null;
            }
        }

        if (mDrawBitmap == null) {
            try {
                mDrawBitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
            } catch (Exception e) {
                e.printStackTrace();
            } catch (OutOfMemoryError e) {
                e.printStackTrace();
            }
        }

        if (mDrawBitmap != null) {
            reset();
            mCanvas.setBitmap(mDrawBitmap);
        }
    }

    private void reset() {
        if (mDrawBitmap != null) {
            mDrawBitmap.eraseColor(Color.TRANSPARENT);
        }
    }
}
```

那么绘制的时候，就改成这样了：

```java
@Override
protected void onDraw(Canvas canvas) {
    if (mCanvasWrapper.isValid()) {
        mCanvasWrapper.save();
        mCanvasWrapper.concat(mMatrix);
        super.onDraw(mCanvasWrapper.getCanvas());
        mCanvasWrapper.restore();
        mCanvasWrapper.draw(canvas);
    } else {
        canvas.save();
        canvas.concat(mMatrix);
        super.onDraw(canvas);
        canvas.restore();
    }
}
```


