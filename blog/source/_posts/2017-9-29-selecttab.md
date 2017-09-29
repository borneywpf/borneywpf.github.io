---
title: Android自定义控件之TabSelectorLayout(微信tab)
date: 2017-09-29 10:09:21
categories: android
tags:
  - android
  - 自定义控件
  - tab
---

老规矩，先看效果图！！！  
{% asset_img tabselectorlayout.gif 效果图 %}

# 概述

天天使用微信，作为一个android developer，当然会对微信的底部tab感兴趣，刚好工作中要开发一个bbs客户端，gui也要求实现一个类似微信底部tab效果的控件，
开始在网上狂搜一通，也找到了类似的东西(至于是哪位大神的，我已经忘了连接，这里就不详述了)，进过自己进一步的封装，自认为完美的实现了gui的设计需求，
然而现在在回想实现的细节，却很难想起其中的关键点，于是乎，决定自己重新写一个，加深自定义ViewGroup和自定义View  
该控件的技术难点：

> 如果通过监听ViewPager的滑动来实现Tab切换的渐变效果

当然对我来说自己重新实现就不止上述一个难点了，下面就让我们还是从源代码中逐步分析如何实现自定义的TabSelectorLayout

# 实现过程

## 上述技术难点如何解决

对于Tab，显示状态只有两个，一个select，一个normal，大家仔细观察微信的渐变效果，其实就是在滑动page的时候，选择的item就是select状态逐渐显示，
而normal状态逐渐消失，而失去select状态的item，与之相反；那么我们就可以通过控制两个状态的drawable的alpha值(状态之间属于"补集")来实现状态切换
的渐变效果

## 定义View的属性

{% codeblock lang:xml %}
<declare-styleable name="TabSelectorLayout">
        <attr name="drawablePadding" format="dimension"/> <!-- drawable和text之间的距离 -->
        <attr name="normalTextColor" format="color"/> <!-- 自然状态下文字颜色 -->
        <attr name="selectTextColor" format="color"/> <!-- 选中状态下文字颜色 -->
        <attr name="android:textSize"/> <!-- 文字大小 -->
    </declare-styleable>
{% endcodeblock %}

## 定义Tab属性类

{% codeblock lang:java %}
public static final class Tab { //TabSelectorLayout嵌套类
        Drawable normalDrawable; //自然状态图片
        Drawable selectDrawable; //选中状态的图片
        String title; //文字
        int position; //tab的位置

        private Tab() { // TabSelectorLayout外部无法创建该类的对象
        }

        Tab setPosition(int position) { //TabSelectorLayout外部无法访问该方法
            this.position = position;
            return this;
        }

        public int getPosition() {
            return position;
        }

        public Tab setTitle(String title) {
            this.title = title;
            return this;
        }

        public Tab setNormalDrawable(Drawable d) {
            normalDrawable = d;
            return this;
        }

        public Tab setSelectDrawable(Drawable d) {
            selectDrawable = d;
            return this;
        }
    }
{% endcodeblock %}

## TabSelectorLayout的属性

{% codeblock lang:java %}
    private ViewPager viewPager; //和关联的ViewPager
    private int textSize = 12; //文字大小
    private int drawablePadding = 10; //drawable和文字之间的距离
    private int normalTextColor, selectTextColor; //文字两种状态的颜色值
{% endcodeblock %}

## 构造方法(解析xml属性)

{% codeblock lang:java %}
public TabSelectorLayout(Context context) {
        super(context);
    }

    public TabSelectorLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TabSelectorLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        TypedArray array = null;
        try {
            array = getContext().getTheme().obtainStyledAttributes(attrs, R.styleable.TabSelectorLayout, defStyleAttr, 0);
            int count = array.getIndexCount();
            for (int i = 0; i < count; i++) {
                final int index = array.getIndex(i);
                if (index == R.styleable.TabSelectorLayout_normalTextColor) {
                    normalTextColor = array.getColor(index, Color.BLACK);
                } else if (index == R.styleable.TabSelectorLayout_selectTextColor) {
                    selectTextColor = array.getColor(index, Color.BLACK);
                } else if (index == R.styleable.TabSelectorLayout_drawablePadding) {
                    drawablePadding = array.getDimensionPixelSize(index, 10);
                } else if (index == R.styleable.TabSelectorLayout_android_textSize) {
                    textSize = array.getDimensionPixelSize(index,
                            (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, textSize, getResources().getDisplayMetrics()));
                }
            }

            Log.d(TAG, "normalTextColor = " + normalTextColor + " drawablePadding = " + drawablePadding + " textSize = " + textSize);
        } finally {
            if (array != null) {
                array.recycle();
            }
        }
    }
{% endcodeblock %}

## 提供获得Tab对象的方法

{% codeblock lang:java %}
public static Tab newTab() {
        return new Tab();
}
{% endcodeblock %}

## 将初始化好的Tab添加到TabSelectorLayout中

{% codeblock lang:java %}
public int addTab(final Tab tab) { //返回添加tab的posion
        //参数判断
        if (tab == null) { 
            throw new NullPointerException("tab is null");
        }
        if (tab.title == null || tab.selectDrawable == null || tab.normalDrawable == null) { 
            throw new IllegalArgumentException("some argument is null");
        }
        final TabView tabView = new TabView(getContext()); //new 出显示的TabView
        tabView.attach(tab); //关联tab到TabView中
        tabView.setOnClickListener(new OnClickListener() { //tab的click监听，切换ViewPager的显示
            @Override
            public void onClick(View v) {
                viewPager.setCurrentItem(tab.getPosition(), false); //切换不滚动
            }
        });
        addView(tabView, new LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.WRAP_CONTENT));
        tab.setPosition(indexOfChild(tabView)); //设置tab的position
        return tab.position;
    }
{% endcodeblock %}

## 重写addView对添加的child做以下限制

{% codeblock lang:java %}
@Override
    public void addView(View child, int index, ViewGroup.LayoutParams params) { //只能添加TabView
        if (child instanceof TabView) {
            super.addView(child, index, params);
        } else {
            throw new IllegalArgumentException("child is not TabView");
        }
    }
{% endcodeblock %}

## 关联ViewPager

{% codeblock lang:java %}
        public void bindViewPager(ViewPager pager) {
        //参数处理
        if (pager == null)
            throw new NullPointerException("pager is null");

        if (viewPager == pager)
            return;

        if (viewPager != null)  //remove对上个ViewPager的监听
            viewPager.removeOnPageChangeListener(pageChangeListener);
        //ViewPager的Adapter数据判读
        PagerAdapter adapter = pager.getAdapter();
        if (adapter == null)
            throw new IllegalArgumentException("pager not set adapter");
        if (adapter.getCount() != getChildCount())
            throw new IllegalArgumentException("pager count is not equeals tab count");

        pager.addOnPageChangeListener(pageChangeListener); //添加OnPageChangeListener

        viewPager = pager;

        setCurrentItem(viewPager.getCurrentItem()); //设置显示的Tab
    }
{% endcodeblock %}

## OnPageChangeListener的实现，给TabView传入ViewPager滑动变化数据

{% codeblock lang:java %}
private ViewPager.OnPageChangeListener pageChangeListener = new ViewPager.SimpleOnPageChangeListener() {
        @Override
        public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            TabView cur = (TabView) getChildAt(position);
            if (positionOffset > 0) {
                cur.setTabOffSet(1 - positionOffset);
                TabView next = (TabView) getChildAt(position + 1);
                next.setTabOffSet(positionOffset);
            } else {
                cur.setTabOffSet(1 - positionOffset);
            }
        }

        @Override
        public void onPageSelected(int position) {
            setCurrentItem(position);
        }
    };
{% endcodeblock %}

## 切换Tab方法

{% codeblock lang:java %}
public void setCurrentItem(int item) {
        final int count = getChildCount();

        for (int i = 0; i < count; i++) {
            TabView child = (TabView) getChildAt(i);
            child.setSelected(item == i); //修改TabView的选中状态
        }
}
{% endcodeblock %}

## 万年不变的onMeasure方法

{% codeblock lang:java %}
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        int width = MeasureSpec.getSize(widthMeasureSpec);
        int height = MeasureSpec.getSize(heightMeasureSpec);

        final int count = getChildCount();

        int childWidth = (width - getPaddingLeft() - getPaddingRight()) / count; //每个child的宽度一样
        int maxChildHeight = 0; //所有child中最高的值

        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                //测量child,精确child的宽度，设定child的最大高度
                child.measure(MeasureSpec.makeMeasureSpec(childWidth, MeasureSpec.EXACTLY), MeasureSpec.makeMeasureSpec(height, MeasureSpec.AT_MOST));
                int h = child.getMeasuredHeight();
                int w = child.getMeasuredWidth();
                maxChildHeight = maxChildHeight > h ? maxChildHeight : h;
            }
        }
        int actionBarHeight = getActionBarHeight(); //默认Layout的高度是ActionBar的高度
        int h = actionBarHeight > maxChildHeight ? actionBarHeight : maxChildHeight;

        setMeasuredDimension(width, h + getPaddingTop() + getPaddingBottom());
    }
{% endcodeblock %}

## 获取actionBar的高度

{% codeblock lang:java %}
private int getActionBarHeight() {
        TypedValue tv = new TypedValue();
        if (getContext().getTheme().resolveAttribute(android.R.attr.actionBarSize, tv, true))
            return TypedValue.complexToDimensionPixelSize(tv.data, getResources().getDisplayMetrics());
        return 0;
    }
{% endcodeblock %}

## 必须实现的onLayout方法

{% codeblock lang:java %}
@Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        final int count = getChildCount();
        int lefPos = getPaddingLeft(); //child的left位置

        final int parentTop = getPaddingTop();
        final int parentBottom = bottom - top - getPaddingBottom();
        final int parentHeight = parentBottom - parentTop;

        final Rect childRect = new Rect(); //child的显示矩阵

        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();

                final int width = child.getMeasuredWidth();
                final int height = child.getMeasuredHeight();

                lefPos += lp.leftMargin; //变化下一个child的left

                childRect.left = lefPos;
                childRect.top = (parentHeight - height) / 2;
                childRect.bottom = childRect.top + height;
                childRect.right = childRect.left + width;

                // Use the child's gravity and size to determine its final
                // frame within its container.
                //Gravity.apply(lp.gravity, width, height, mTmpContainerRect, mTmpChildRect);

                child.layout(childRect.left, childRect.top, childRect.right, childRect.bottom); //layout child

                lefPos += width; //变化下一个child的left
                lefPos += lp.rightMargin; //变化下一个child的left
            }
        }
{% endcodeblock %}

## 定义TabView

{% codeblock lang:java %}
private class TabView extends View { //私有内部类
{% endcodeblock %}

## TabView属性

{% codeblock lang:java %}
        private int normalAlpha = 255;  //自然状态下的alpha
        private int viewWidth; //TabView宽度
        private int viewHeight;//TabView高度
        private final Paint textNormalPaint = new Paint(Paint.ANTI_ALIAS_FLAG | Paint.SUBPIXEL_TEXT_FLAG); //自然状态text画笔
        private final Paint textSelectPaint = new Paint(Paint.ANTI_ALIAS_FLAG | Paint.SUBPIXEL_TEXT_FLAG);//选中状态text画笔
        private final Rect boundText = new Rect(); //text的bound
        private Tab tab; //关联的tab
{% endcodeblock %}

## TabView属性初始化

{% codeblock lang:java %}
private void initText() {
            textNormalPaint.setColor(normalTextColor);
            textNormalPaint.setTextSize(textSize);

            textSelectPaint.setColor(selectTextColor);
            textSelectPaint.setTextSize(textSize);
        }
{% endcodeblock %}

## 获得指定字体大小text的显示矩阵

{% codeblock lang:java %}
private void measureText() {
            textNormalPaint.getTextBounds(tab.title, 0, tab.title.length(), boundText);
}
{% endcodeblock %}

## 通过监听到的ViewPager滑动变化来修改显示的alpha

{% codeblock lang:java %}
void setTabOffSet(float offSet) {
            normalAlpha = (int) (255 - offSet * 255);
            invalidate();
 }
{% endcodeblock %}

## 重新setSelected方法，来修改显示的alpha

{% codeblock lang:java %}
@Override
        public void setSelected(boolean selected) {
            normalAlpha = selected ? 0 : 255;
            super.setSelected(selected);
        }
{% endcodeblock %}

## TabView的onMeasure方法

{% codeblock lang:java %}
@Override
        protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            int width = MeasureSpec.getSize(widthMeasureSpec);
            int widthMode = MeasureSpec.getMode(widthMeasureSpec);

            int height = MeasureSpec.getSize(heightMeasureSpec);
            int heightMode = MeasureSpec.getMode(heightMeasureSpec);
            int w = 0, h = 0;

            measureText();//获得指定字体大小text的显示矩阵
            //显示内容的最大宽度
            int contentWidth = Math.max(boundText.width(), Math.max(tab.normalDrawable.getIntrinsicWidth(), tab.selectDrawable.getIntrinsicWidth()));
            //期望的宽度
            int desiredWidth = getPaddingLeft() + getPaddingRight() + contentWidth;
            switch (widthMode) {
                case MeasureSpec.AT_MOST:
                    w = Math.min(width, desiredWidth);
                    break;
                case MeasureSpec.EXACTLY: //实现的ViewGroup中已经指定精确TabView的宽度
                    w = width;
                    break;
                case MeasureSpec.UNSPECIFIED:
                    w = desiredWidth;
                    break;
            }
            //获得显示内容的高度，注意这里计算text的高度使用了Paint的getFontSpacing()方法，原因可以参考文后资料
            int contentHeight = (int) (textNormalPaint.getFontSpacing() + Math.max(tab.normalDrawable.getIntrinsicHeight(), tab.selectDrawable.getIntrinsicHeight()) + drawablePadding);
            //期望的最大高度
            int desiredHeight = getPaddingTop() + getPaddingBottom() + contentHeight;
            switch (heightMode) {
                case MeasureSpec.AT_MOST:
                    h = Math.min(height, desiredHeight);
                    break;
                case MeasureSpec.EXACTLY:
                    h = height;
                    break;
                case MeasureSpec.UNSPECIFIED:
                    h = height;
                    break;
            }
            setMeasuredDimension(w, h);
            viewWidth = getMeasuredWidth();
            viewHeight = getMeasuredHeight();
        }
{% endcodeblock %}

## TabView的onDraw方法

{% codeblock lang:java %}
@Override
        protected void onDraw(Canvas canvas) {
            super.onDraw(canvas);
            drawNoramalBitmap(canvas);
            drawSelectBitmap(canvas);
            drawText(canvas);
        }
{% endcodeblock %}

## TabView的自然状态drawable的绘制

{% codeblock lang:java %}
private void drawNoramalBitmap(Canvas canvas) {
            int width = tab.normalDrawable.getIntrinsicWidth();
            int height = tab.normalDrawable.getIntrinsicHeight();
            int left = (viewWidth - width) / 2;
            int top = (viewHeight - height - boundText.height() - drawablePadding) >> 1;
            tab.normalDrawable.setBounds(left, top, left + width, top + height);
            tab.normalDrawable.setAlpha(normalAlpha); //修改绘制drawable的alpha
            tab.normalDrawable.draw(canvas);
        }
{% endcodeblock %}

## TabView的选中状态drawable的绘制

{% codeblock lang:java %}
private void drawSelectBitmap(Canvas canvas) {
            int width = tab.selectDrawable.getIntrinsicWidth();
            int height = tab.selectDrawable.getIntrinsicHeight();
            int left = (viewWidth - width) / 2;
            int top = (viewHeight - height - boundText.height() - drawablePadding) >> 1;
            tab.selectDrawable.setBounds(left, top, left + width, top + height);
            tab.selectDrawable.setAlpha(255 - normalAlpha); //修改绘制drawable的alpha
            tab.selectDrawable.draw(canvas);
        }
{% endcodeblock %}

## TabView的text的绘制

{% codeblock lang:java %}
private void drawText(Canvas canvas) {
            int drawableHeight = Math.max(tab.normalDrawable.getIntrinsicHeight(), tab.selectDrawable.getIntrinsicHeight());
            float x = (viewWidth - boundText.width()) / 2.0f;
            float y = (viewHeight + drawableHeight + boundText.height() + drawablePadding) >> 1;
            textNormalPaint.setAlpha(normalAlpha); //修改绘制text自然状态的alpha
            canvas.drawText(tab.title, x, y, textNormalPaint);
            textSelectPaint.setAlpha(255 - normalAlpha); //修改绘制text选中状态的alpha
            canvas.drawText(tab.title, x, y, textSelectPaint);
        }
{% endcodeblock %}

# 如何使用

## 使用的xml代码

{% codeblock lang:xml %}
<com.think.android.widget.TabSelectorLayout
        android:id="@+id/tabselectorlayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        app:selectTextColor="@android:color/holo_green_dark"
        app:normalTextColor="@android:color/darker_gray"
        app:drawablePadding="5dp"
        android:textSize="13sp"/>
{% endcodeblock %}

## 使用的java代码

{% codeblock lang:java %}
//首先获得TabSelectorLayout对象
mTabSelectorLayout = (TabSelectorLayout) findViewById(R.id.tabselectorlayout);
...
//添加tab
mTabSelectorLayout.addTab(TabSelectorLayout.newTab()
                .setNormalDrawable(resources.getDrawable(R.drawable.ic_tab_moment))
                .setSelectDrawable(resources.getDrawable(R.drawable.ic_tab_moment_select))
                .setTitle("moment"));
...
//和ViewPager绑定
mTabSelectorLayout.bindViewPager(mViewPager);
{% endcodeblock %}

# 文章感想
写完本文就觉得自己啰嗦了，总想将所有的内容都写出来，哪怕再简单的东西，大量的代码对读者来说都很简单，但我只想展示自己实现这个控件一步步的过程，
期望自己以后的文章尽量提升blog的水平吧  

到这里本文就到此结束了.[[示例代码]](https://github.com/borneywpf/ThinkAndroid)

------

参考资料  
如何测量text的高度  
[[资料1](http://stackoverflow.com/questions/3654321/measuring-text-height-to-be-drawn-on-canvas-android)]  
[[资料2](http://jyhshin.pixnet.net/blog/post/37074598-android-get-text-height)]
