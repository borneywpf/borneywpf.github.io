---
title: Android自定义控件之CircleImageView
date: 2017-09-29 10:26:55
categories: android
tags:
  - android
  - 自定义控件
  - imageview
---

最近写代码需要定义一个显示圆形图片的控件，这种东西网上有很多，千篇一律的方式就是通过Circle和Bitmap取交集的方式，这种方式实现时有严重的锯齿问题；
第二种是通过Paint设置Xfermode来实现的，这种方式可以很好的屏蔽锯齿，但是在使用的过程中定义的Circle必须和要显示的位图一样大小，如果要显示位图的一
部分就不可以了；另外一种认为是比较优化的方式是通过BitmapShaper渲染来实现的，而且绘过程也比较简单；

下面就是我自己实现的CircleImageView，做了进一步的优化：

> - 通过使用Layer解决了Paint设置Xfermode不能绘制图片局部的问题
> - 可以绘制圆角图片
> - 可以选择使用Xfermode或者BitmapShaper的方式来渲染图片

通过测试发现使用Xfermode的方式是比BitmapShaper更优一些，首先是绘制效率，小图没有什么，当大图的时候Xfermode就稍比BitmapShaper快一点，而且使用
Xfermode生成的图片比BitmapShaper生成的图片要小，我测试的过程中，同样的效果图Xfermode生成的图片大小是859kb，而BitmapShaper生成的图片是881kb，
所以我个人比较倾向使用Xfermode的方式，读者可以自己测试以下，下面我们就来看看具体的实现过程吧。

{% asset_img android_circleimageview_1.png 效果图 %}

# 源代码

## 属性文件配置

{% codeblock attrs.xml lang:xml %}
<declare-styleable name="CircleImageView">
        <attr name="radius" format="dimension"/>
        <attr name="iscircle" format="boolean"/>
        <attr name="cropType">
            <flag name="leftTop" value="1"/>
            <flag name="leftBottom" value="2"/>
            <flag name="rightTop" value="3"/>
            <flag name="rightBottom" value="4"/>
            <flag name="leftCenter" value="5"/>
            <flag name="rightCenter" value="6"/>
            <flag name="topCenter" value="7"/>
            <flag name="bottomCenter" value="8"/>
            <flag name="center" value="9"/>
        </attr>
        <attr name="drawType">
            <flag name="shader" value="1"/>
            <flag name="xfermode" value="2"/>
        </attr>
    </declare-styleable>
{% endcodeblock %}

## CircleImageView

{% codeblock CircleImageView.java lang:xml %}
public class CircleImageView extends ImageView {
    private static final String TAG = "CircleImageView";

    private Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);

    /**
     * 半径
     */
    private int radius = 0;

    /**
     * 是否是圆形裁剪
     * true圈形，false圆角图片
     */
    private boolean isCircle = true;

    /**
     * 裁剪类型
     */
    private int cropType = CropType.CENTER;

    /**
     * 绘制类型
     */
    private int drawType = DrawType.XFERMODE;

    /**
     * 圈形图片的裁剪位置
     */
    public static final class CropType {
        public static final int LEFT_TOP = 1; //显示图片的左上角部分
        public static final int LEFT_BOTTOM = 2;//显示图片的左下角部分
        public static final int RIGHT_TOP = 3;//显示图片的右上角部分
        public static final int RIGHT_BOTTOM = 4;//显示图片的右下角部分
        public static final int LEFT_CENTER = 5;//显示图片的左居中部分
        public static final int RIGHT_CENTER = 6;//显示图片的右居中部分
        public static final int TOP_CENTER = 7;//显示图片的上居中部分
        public static final int BOTTOM_CENTER = 8;//显示图片的下居中部分
        public static final int CENTER = 9;//显示图片的居中部分
    }

    /**
     * 使用BitmapShaper方式绘制还是Xfermode的方式绘制
     */
    public static final class DrawType {
        public static final int SHADER = 1; //使用BitmapShaper方式绘制
        public static final int XFERMODE = 2; //使用Xfermode方式绘制
    }

    public CircleImageView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        TypedArray array = null;
        try {
            array = context.getTheme().obtainStyledAttributes(attrs, R.styleable.CircleImageView, 0, 0);
            int count = array.getIndexCount();
            Log.d(TAG, "count = " + count);
            for (int i = 0; i < count; i++) {
                final int index = array.getIndex(i);
                if (index == R.styleable.CircleImageView_radius) {
                    radius = array.getDimensionPixelSize(index, 0); //初始化半径
                } else if (index == R.styleable.CircleImageView_cropType) {
                    cropType = array.getInteger(index, CropType.CENTER); //初始化截取类型
                } else if (index == R.styleable.CircleImageView_drawType) {
                    drawType = array.getInteger(index, DrawType.XFERMODE); //初始化绘制类型
                } else if (index == R.styleable.CircleImageView_iscircle) {
                    isCircle = array.getBoolean(index, true); //是否绘制圆
                }
            }
            Log.d(TAG, "radius = " + radius + " cropType = " + cropType + " drawType = " + drawType + " isCircle = " + isCircle);
        } finally {
            if (array != null) {
                array.recycle();
            }
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int intrinsicWidth = getDrawable().getIntrinsicWidth();
        int intrinsicHeight = getDrawable().getIntrinsicHeight();
        Log.d(TAG, "intrinsicWidth = " + intrinsicWidth + " intrinsicHeight = " + intrinsicHeight);
        if (isCircle) {
            /**
            *1、如果圆形半径设置为0，则使用图片中宽高之间的最小值作为圆形的半径
            *2、如果圆形半径不为0，则取半径，宽，高之间的最小值作为半径
            **/
            int width = resolveAdjustedSize(radius == 0 ? intrinsicWidth : Math.min(intrinsicWidth, radius * 2), Integer.MAX_VALUE, widthMeasureSpec);
            int height = resolveAdjustedSize(radius == 0 ? intrinsicHeight : Math.min(intrinsicHeight, radius * 2), Integer.MAX_VALUE, heightMeasureSpec);

            int border = Math.min(width, height);

            radius = border / 2;

            Log.d(TAG, "isCircle border = " + border + " radius = " + radius);
            setMeasuredDimension(border, border);
        } else {
            /**
            *圆角图片的圆角半径取半径，宽，高之间的最小值
            **/
            int width = resolveAdjustedSize(intrinsicWidth, Integer.MAX_VALUE, widthMeasureSpec);
            int height = resolveAdjustedSize(intrinsicHeight, Integer.MAX_VALUE, heightMeasureSpec);
            radius = Math.min(Math.min(width, height), radius);
            Log.d(TAG, "isCircle not border = " + Math.min(width, height) + " radius = " + radius);
            setMeasuredDimension(width, height);
        }
    }

    /**
    *设置半径
    **/
    public void setRadius(int radius) {
        this.radius = radius;
        requestLayout();
    }

    /**
    *设置截取类型
    **/
    public void setCropType(int cropType) {
        this.cropType = cropType;
        invalidate();
    }

    /**
    *设置绘制类型
    **/
    public void setDrawType(int drawType) {
        this.drawType = drawType;
        invalidate();
    }

    /**
    *此方法参考了ImageView的resolveAdjustedSize方法，可以自己查阅
    **/
    private int resolveAdjustedSize(int desiredSize, int maxSize,
                                    int measureSpec) {
        int result = desiredSize;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
        switch (specMode) {
            case MeasureSpec.UNSPECIFIED:
                /* Parent says we can be as big as we want. Just don't be larger
                   than max size imposed on ourselves.
                */
                result = Math.min(desiredSize, maxSize);
                break;
            case MeasureSpec.AT_MOST:
                // Parent says we can be as big as we want, up to specSize.
                // Don't be larger than specSize, and don't be larger than
                // the max size imposed on ourselves.
                result = Math.min(Math.min(desiredSize, specSize), maxSize);
                break;
            case MeasureSpec.EXACTLY:
                // No choice. Do what we are told.
                result = specSize;
                break;
        }
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        //super.onDraw(canvas);
        if (getDrawable() == null) {
            return;
        }
        if (drawType == DrawType.XFERMODE) {
            drawByXfermode(canvas); //使用Xfermode方式绘制
        } else {
            drawByShader(canvas); //使用Bitmap方式绘制
        }
    }

    private void drawByXfermode(Canvas canvas) {
        int width = getWidth();
        int height = getHeight();
        int restore = canvas.saveLayer(0, 0, width, height, null,
            Canvas.ALL_SAVE_FLAG);  //保存Layer
        if (isCircle) {
            canvas.drawCircle(radius, radius, radius, paint); //绘制圆形
            paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN)); //设置Xfermode
            canvas.saveLayer(0, 0, width, height, paint, Canvas.ALL_SAVE_FLAG); //二次保存Layer
            Bitmap bitmap = drawableToBitmap(getDrawable());
            int[] xy = getCropTypeCircleXY(bitmap); //获取图片中显示圆形的中心坐标
            Rect src = new Rect(); //定义设置源图片的显示区域
            src.left = xy[0] - radius;
            src.right = xy[0] + radius;
            src.top = xy[1] - radius;
            src.bottom = xy[1] + radius;
            canvas.drawBitmap(bitmap, src, new Rect(0, 0, width, height), null); //绘制图片
        } else {
            RectF rect = new RectF(0, 0, width, height);
            canvas.drawRoundRect(rect, radius, radius, paint); //绘制圆角矩形
            paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));//设置Xfermode
            canvas.saveLayer(0, 0, width, height, paint, Canvas.ALL_SAVE_FLAG);//二次保存Layer
            Bitmap bitmap = drawableToBitmap(getDrawable());
            canvas.drawBitmap(bitmap, 0, 0, null);
        }
        canvas.restoreToCount(restore); //恢复Layer
        paint.setXfermode(null);
    }

    private void drawByShader(Canvas canvas) {
        Bitmap src = drawableToBitmap(getDrawable());
        if (isCircle) {
            int[] cropTypeCircleXY = getCropTypeCircleXY(src);//获取图片中显示圆形的中心坐标
            Bitmap bitmap = Bitmap.createBitmap(src, cropTypeCircleXY[0] - radius, cropTypeCircleXY[1] - radius, getWidth(), getHeight()); //创建显示区域的图片
            BitmapShader bitmapShader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP); //设置BitmapShader
            paint.setShader(bitmapShader);
            canvas.drawCircle(radius, radius, radius, paint);
        } else {
            BitmapShader bitmapShader = new BitmapShader(src, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);
            paint.setShader(bitmapShader);
            int width = getWidth();
            int height = getHeight();
            RectF rect = new RectF(0, 0, width, height);
            canvas.drawRoundRect(rect, radius, radius, paint);
        }
    }

    /**
    *将drawable转换成bitmap
    **/
    private Bitmap drawableToBitmap(Drawable drawable) {
        int w = drawable.getIntrinsicWidth();
        int h = drawable.getIntrinsicHeight();
        Bitmap bitmap = Bitmap.createBitmap(w, h, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        drawable.draw(canvas);
        return bitmap;
    }

    /**
    *获得显示区域的圆形坐标
    **/
    private int[] getCropTypeCircleXY(Bitmap bitmap) {
        int width = bitmap.getWidth();
        int height = bitmap.getHeight();
        int[] xy;
        switch (cropType) {
            case CropType.LEFT_TOP:
                xy = new int[]{radius, radius};
                break;
            case CropType.LEFT_BOTTOM:
                xy = new int[]{radius, height - radius};
                break;
            case CropType.RIGHT_TOP:
                xy = new int[]{width - radius, radius};
                break;
            case CropType.RIGHT_BOTTOM:
                xy = new int[]{width - radius, height - radius};
                break;
            case CropType.LEFT_CENTER:
                xy = new int[]{radius, height / 2};
                break;
            case CropType.RIGHT_CENTER:
                xy = new int[]{width - radius, height / 2};
                break;
            case CropType.TOP_CENTER:
                xy = new int[]{width / 2, radius};
                break;
            case CropType.BOTTOM_CENTER:
                xy = new int[]{width / 2, height - radius};
                break;
            case CropType.CENTER:
                xy = new int[]{width / 2, height / 2};
                break;
            default:
                xy = new int[]{width / 2, height / 2};
                break;
        }
        return xy;
    }
}
{% endcodeblock %}

# 使用方法

{% codeblock 显示的xml lang:xml %}
<com.think.android.widget.CircleImageView
                    xmlns:android="http://schemas.android.com/apk/res/android"
                    xmlns:app="http://schemas.android.com/apk/res-auto"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:src="@drawable/circleimageview_demo"
                    android:background="@android:color/darker_gray"
                    app:radius="60dp"
                    app:cropType="leftTop"
                    app:drawType="shader"/>
{% endcodeblock %}

本文[[示例代码]](https://github.com/borneywpf/ThinkAndroid)

---
参考资料  
[CircleImageView-方式2](http://www.jianshu.com/p/4cdfb546e381)
