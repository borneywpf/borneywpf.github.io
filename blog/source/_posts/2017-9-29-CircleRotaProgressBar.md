---
title: Android自定义控件之CircleRotaProgressBar
date: 2017-09-29 13:34:39
categories: android
tags:
  - android
  - 自定义控件
  - imageview
---

先看效果图！！！
{% asset_img circlerotaprogressbar.gif 效果图 %}

# 控件前因
以前在工作中，GUI设计了一个上图的进度条效果，很显然android原生没有这样的控件，在github等开源社区溜达了一圈也没有找到类似的实现，无奈之余，只能自定义了，当时能力有限，实现控件时遇到了几个当时认为的技术难点；

>- 绘制进度的的弧度角度（这种现在来说也不是什么事情了）
>- 弧度如何实现圆头(当时我还傻呼呼的在弧度两端分别绘制一个圆，以为还是不错的实现)
>- 图中两个点的动画变减速效果(当时对属性动画理解的还不到位，自己觉得这个还是一个蛮高深的东西，于是自己花了一天时间专门研究了android动画对时间差值器([TimeInterpolator](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/android/animation/TimeInterpolator.java#TimeInterpolator))的运用原理，当时蛮有成就感的，也学到了一些东西)

# 重新的封装
这几天回顾了下自己实现的这个控件，虽然当时实现的有些复杂，但我还蛮喜欢这个控件的效果；于是乎就查询了些资料，重新封装，顺便温习下一些知识点，也希望能对读者有用.  
下面我会逐步分析重新实现控件的内容

## 自定义属性

{% codeblock attrs.xml lang:xml %}
<declare-styleable name="CircleRotaProgressBar">
        <attr name="progressWidth" format="dimension"/> <!-- 进度条的宽度 -->
        <attr name="secondaryProgressWidth" format="dimension"/> <!-- 进度条底圆的宽度 -->
        <attr name="duration" format="integer"/> <!-- 动画持续时间 -->
        <attr name="secondaryAnimatorDelay" format="integer"/> <!-- 第二个点动画启动延时时间 -->
        <attr name="progressColor" format="color"/> <!-- 进度条的颜色 -->
        <attr name="secondaryProgressColor" format="color"/> <!-- 进度条底圆的颜色 -->
    </declare-styleable>
{% endcodeblock %}

## 基于TextView实现

{% codeblock lang:java %}
public class CircleRotaProgressBar extends TextView {...}
{% endcodeblock %}

定义需要的变量
{% codeblock lang:java %}
/**
     * 进度
     */
    private int progress = 0;
    /**
     * 进度条宽度
     */
    private float progressWidth = 4.0f;
    /**
     * 进度条宽度的一半
     */
    private float halfProgressWidth = progressWidth / 2;
    /**
     * 进度条底圆的宽度
     */
    private float secondaryProgressWidth = 2.0f;
    /**
     * 进度条颜色
     */
    private int progressColor = Color.argb(255, 255, 255, 255);
    /**
     * 进度条底圆的颜色
     */
    private int secondaryProgressColor = Color.argb(150, 255, 255, 255);
    /**
     * 动画持续时间
     */
    private int duration = 1500;
    /**
     * 第二个点动画开始延时时间
     */
    private int secondaryAnimatorDelay = 200;
    /**
     * 进度条弧度
     */
    private float sweepAngle;
    /**
     * 动画差值器
     */
    private TimeInterpolator interpolator = new DecelerateInterpolator();
    /**
     * 第一个点动画通过属性动画中{@link PropertyValuesHolder}的方式实现
     */
    private PropertyValuesHolder pvh;
    /**
     * 定义两个点的属性动画
     */
    private ValueAnimator valueAnimator, _valueAnimator;
    /**
     * 初始状态
     */
    private State state = State.PAUSE;

    private Paint paint = new Paint();
{% endcodeblock %}

## 定义了进度动画的状态

{% codeblock lang:java %}
    /**
     * 定义进度条的两个动画状态
     */
    public enum State {
        START, //开始动画

        PAUSE; //结束动画
    }
{% endcodeblock %}

## 构造方法

{% codeblock lang:java %}
public CircleRotaProgressBar(Context context) {
        this(context, null);
    }

    public CircleRotaProgressBar(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleRotaProgressBar(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray array = null;

        try {
            array = context.getTheme().obtainStyledAttributes(attrs, R.styleable.CircleRotaProgressBar, defStyleAttr, 0);
            int count = array.getIndexCount();
            for (int i = 0; i < count; i++) {
                final int index = array.getIndex(i);
                if (index == R.styleable.CircleRotaProgressBar_progressWidth) {
                    progressWidth = array.getDimensionPixelSize(index, 0);
                    halfProgressWidth = progressWidth / 2;
                } else if (index == R.styleable.CircleRotaProgressBar_secondaryProgressWidth) {
                    secondaryProgressWidth = array.getDimensionPixelSize(index, 0);
                } else if (index == R.styleable.CircleRotaProgressBar_progressColor) {
                    progressColor = array.getColor(index, progressColor);
                } else if (index == R.styleable.CircleRotaProgressBar_secondaryProgressColor) {
                    secondaryProgressColor = array.getColor(index, secondaryProgressColor);
                } else if (index == R.styleable.CircleRotaProgressBar_duration) {
                    duration = array.getInt(index, duration);
                } else if (index == R.styleable.CircleRotaProgressBar_secondaryAnimatorDelay) {
                    secondaryAnimatorDelay = array.getInt(index, secondaryAnimatorDelay);
                }
            }
            if (secondaryProgressWidth >= progressWidth) {
                throw new RuntimeException("secondaryProgressWidth(" + secondaryProgressWidth + ") must less than progressWidth(" + progressWidth + ")");
            }
        } finally {
            if (array != null)
                array.recycle();
        }
        setGravity(Gravity.CENTER);
        init();
    }
{% endcodeblock %}

## 初始化方法

{% codeblock lang:java %}
private void init() {
        sweepAngle = getSweepAngleByProgress(progress);
        pvh = PropertyValuesHolder.ofFloat("sweepAngle", sweepAngle, -360.0f);
        valueAnimator = ValueAnimator.ofPropertyValuesHolder(pvh);
        _valueAnimator = ValueAnimator.ofFloat(sweepAngle, -360.0f);

        paint.setFlags(Paint.ANTI_ALIAS_FLAG | Paint.FILTER_BITMAP_FLAG);
        paint.setStrokeCap(Paint.Cap.ROUND); //画笔是圆形模式(这里很轻松的解决掉我之前的问题二)
        paint.setDither(true);

        valueAnimator.setInterpolator(interpolator);
        valueAnimator.setDuration(duration);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) { //属性动画更新时绘制
                invalidate();
            }
        });

        _valueAnimator.setInterpolator(interpolator);
        _valueAnimator.setDuration(duration);
        _valueAnimator.setStartDelay(secondaryAnimatorDelay);//第二个点动画延时
        _valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                if (!valueAnimator.isRunning()) { //因为第一个动画running的过程中已经调用了invalidate方法，这里判读就是避免重复调用绘制
                    invalidate();
                }
            }
        });
        _valueAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                if (!valueAnimator.isStarted() && state == State.START) { //在第二个点结束动画后，才继续开始第一个动画和第二个动画
                   startAnimation();
                }
            }
        });
    }
{% endcodeblock %}

## 既然是进度条控件，自然要能更新进度

{% codeblock lang:java %}
    /**
     * 设置进度
     * @param p
     */
    public void setProgress(int p) {
        if (p < 0 || p > 100) {
            throw new RuntimeException("progress(" + p + ") must be betain 0 and 100");
        }
        if (p == progress) {
            return;
        }
        progress = p;
        sweepAngle = getSweepAngleByProgress(progress);
        pvh.setFloatValues(sweepAngle, -360.0f);
        _valueAnimator.setFloatValues(sweepAngle, -360.0f);
        setText(String.valueOf(p) + "%");
    }
{% endcodeblock %}

## 如何用进度转化成需要绘制的弧度呢？

{% codeblock lang:java %}
    /**
     * 根据进度获得绘制进度的弧度
     * @param progress
     * @return
     */
    private float getSweepAngleByProgress(int progress) {
        return (progress / 100.0f) * 360 * -1;
    }
{% endcodeblock %}

## 既然由动画状态State，那么怎么少的了更新状态的方法

{% codeblock lang:java %}
/**
     * 设置进度运行状态
     * @param s
     */
    public void setState(State s) {
        if (s == null) {
            throw new NullPointerException("s is null");
        }
        if (s == state) {
            return;
        }
        state = s;
        if (state == State.START) {
            startAnimation();
        } else {
            endAnimation();
        }
    }
{% endcodeblock %}

## 覆写了setVisibility方法，提升性能

{% codeblock lang:java %}
@Override
    public void setVisibility(int visibility) {
        if (visibility != getVisibility()) {
            super.setVisibility(visibility);
            if (visibility == GONE || visibility == INVISIBLE) {
                endAnimation();
            } else {
                if (state == State.START) {
                    startAnimation();
                } else {
                    endAnimation();
                }
            }
        }
    }
{% endcodeblock %}

## 怎么少的了覆写onMeasure方法呢

{% codeblock lang:java %}
/**
     * 重写onMeasure方法，保证是一个正方形
     * @param widthMeasureSpec
     * @param heightMeasureSpec
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int size = 200;
        int w = MeasureSpec.getSize(widthMeasureSpec);
        w = (w > 0) ? w : size;
        int h = MeasureSpec.getSize(heightMeasureSpec);
        h = (h > 0) ? w : size;

        Log.d(TAG, "w = " + w + " h = " + h);

        w += getPaddingLeft() + getPaddingRight();
        h += getPaddingTop() + getPaddingBottom();

        Log.d(TAG, "p w = " + w + " h = " + h);

        final int measuredWidth = resolveSizeAndState(Math.min(w, h), widthMeasureSpec, 0);
        final int measuredHeight = resolveSizeAndState(Math.min(w, h), heightMeasureSpec, 0);

        setMeasuredDimension(measuredWidth, measuredHeight);
    }
{% endcodeblock %}

## 关键的onDraw方法

{% codeblock lang:java %}
@Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        int width = getWidth();
        int height = getHeight();

        int paddingLeft = getPaddingLeft();
        int paddingRight = getPaddingRight();
        int paddingTop = getPaddingTop();
        int paddingBottom = getPaddingBottom();

        /**
         * 进度圆的中心坐标
         */
        float cx = paddingLeft + (width - paddingLeft - paddingRight) / 2.0f;
        float py = paddingTop + (height - paddingTop - paddingBottom) / 2.0f;

        paint.setStyle(Paint.Style.STROKE);

        /**
         * 绘制进度条底圆
         */
        paint.setColor(secondaryProgressColor);
        paint.setStrokeWidth(secondaryProgressWidth);
        canvas.drawCircle(cx, py, (width - paddingLeft - paddingRight) / 2.0f - halfProgressWidth, paint);

        /**
         * 绘制进度条
         */
        paint.setColor(progressColor);
        RectF rectF = new RectF(halfProgressWidth + paddingLeft,
                halfProgressWidth + paddingTop,
                width - halfProgressWidth - paddingRight,
                height - halfProgressWidth - paddingBottom);
        paint.setStrokeWidth(progressWidth);
        canvas.drawArc(rectF, -90.0f, sweepAngle, false, paint);

        paint.setStyle(Paint.Style.FILL);

        /**
         * 绘制第一个动画点
         */
        canvas.save();
        canvas.rotate(((Float) valueAnimator.getAnimatedValue("sweepAngle")).floatValue(), cx, py);
        canvas.drawPoint(cx, halfProgressWidth + paddingTop, paint);
        canvas.restore();

        /**
         * 绘制第二个动画点
         */
        canvas.save();
        canvas.rotate(((Float) _valueAnimator.getAnimatedValue()).floatValue(), cx, py);
        canvas.drawPoint(cx, halfProgressWidth + paddingTop, paint);
        canvas.restore();
    }
{% endcodeblock %}

## 简单的开始和结束动画方法

{% codeblock lang:java %}
/**
     * 开始动画
     */
    private void startAnimation() {
        valueAnimator.start();
        _valueAnimator.start();
    }

    /**
     * 结束动画
     */
    private void endAnimation() {
        valueAnimator.end();
        _valueAnimator.end();
        valueAnimator.cancel();
        _valueAnimator.cancel();
    }
{% endcodeblock %}

# 定义了几个默认style供大家使用！！！

{% codeblock lang:xml %}
<style name="Widget.CircleRotaProgressBarNormal" parent="@android:style/Widget">
        <item name="progressWidth">6dp</item>
        <item name="secondaryProgressWidth">3dp</item>
    </style>
    <style name="Widget.CircleRotaProgressBarWide" parent="@android:style/Widget">
        <item name="progressWidth">8dp</item>
        <item name="secondaryProgressWidth">4dp</item>
    </style>
    <style name="Widget.CircleRotaProgressBarNarrow" parent="@android:style/Widget">
        <item name="progressWidth">4dp</item>
        <item name="secondaryProgressWidth">2dp</item>
    </style>
{% endcodeblock %}

# 简单的使用示例

{% codeblock lang:xml %}
<com.think.android.widget.CircleRotaProgressBar
            android:id="@+id/circlerotaprogressbar2"
            android:layout_width="180dp"
            android:layout_height="180dp"
            android:layout_gravity="center"
            android:layout_centerHorizontal="true"
            android:padding="10dp"
            android:layout_marginTop="20dp"
            android:textSize="46sp"
            android:textColor="@android:color/white"
            android:background="@android:color/holo_blue_light"
            android:layout_below="@id/circlerotaprogressbar1"
            style="@style/Widget.CircleRotaProgressBarWide"/>
{% endcodeblock %}

到这里本文就到此结束了.[[示例代码]](https://github.com/borneywpf/ThinkAndroid)
