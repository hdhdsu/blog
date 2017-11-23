<!--
author: 冷火-王胜 
date: 2017-2-28 
title: 自定义View学习总结(一)
tags: Android View
category: Android
status: publish 
-->
#####1.为什么会引入自定义view
现有的控件不满足我们的需求，或者项目中频繁用到一种需要我们自定义的控件，把它封装起来提高复用性。
#####2.自定义view的使用
######2.1 自定义控件的绘制流程：

    1,准备工作:(加载阶段)
    2,规划大小:(测量阶段)
    3,绘制位置:(布局阶段)
    4,画:(绘制阶段)
1.定义一个类继承View或View的子类

    public class MyView extends View {
      ...
    }
2.重写构造方法

    //当需要new当前视图的时候需要重写这个构造
	public MyView(Context context) {
    //一般处理为让其调用含有两个参数的构造方法
		this(context, null);
	}
    //当需要将自定义View声明在xml文件中使用的时候需要重写这个构造方法
	public MyView(Context context, AttributeSet attrs) {
    //一般处理为调用第三个构造
		this(context, attrs, 0);
	}
    //当需要将自定义View声明在xml文件中,并且当前控件需要自定义主题样式的时候.重写这个构造方法
	public MyView(Context context, AttributeSet attrs, int defStyleAttr) {
		super(context, attrs, defStyleAttr);
    //做初始化操作
		init();
	}
构造方法一般用于数据的初始化加载
开发中也存在只重写两个参数的构造方法的情况,但这样做就不能再去直接new控件了.

3.重写onMeasure
onMeasure在自定义view中有时候不用重写，因为如果是自定义view的话在xml声明中给定的是确定的数值或者match_parent时不用重写该方法，但是如果给定的数值时wrap_content的时候就需要重写了，因为此时view不知道采用哪个大小。但是在自定义ViewGroup的时候要重写onMeasure方法。

onMeasure()方法:用于测量控件,如果控件还有子控件,则会递归测量子控件
如果是ViewGroup,默认只会测量自身控件

    MeasureSpec:是由父布局和子控件共同决定的,是一个组合值,用于控件的宽高
    MeasureSpec是一个32位int值
    MeasureSpec是由两部分组成:
    mode:控件尺寸的模式,前两位
    size:控件的尺寸,后三十位


其中mode分为三种:
1,UNSPECIFIED:数值为0<<30
未限定实际宽高,这种情况较少,例如:ScrollView对于其子视图的高度的限定
2,EXACTLY:数值为1<<30
明确的尺寸,例如:xxdp,MATCH_PARENT
3,AT_MOST:数值为2<<30
至多为多少,例如WRAP_CONTENT,若控件本身没有默认尺寸,则系统会尽可能的把空间赋予控件,为MATCH_PARENT

常用api

    setMeasureDimension(width,height):设置控件的最终尺寸
    MeasureSpec spec=  MeasureSpec.makeMeasureSpec(size,mode);用于指定MeasureSpec
    MeasureSpec.getSize(measureSpec);通过MeasureSpec获取size
    MeasureSpec.getMode(measureSpec);通过MeasureSpec获取mode
    getSuggestedMinimumWidth():获得背景图的宽度,如果没有背景,则返回值为0
4.onDraw 绘制
onDraw里面只有一个参数Canvas canvas，这个参数是用来绘制自定义View的，这个方法里面还有一个重要的变量，Paint paint,这个变量是用来给设置画笔的

（Paint画笔相关api）

    //给Paint设置颜色
    paint.setColor(Color.RED);
    //用此Paint绘制出的图形没有锯齿,但会损耗性能
    paint.setAntiAlias(true);
    //设置Paint样式(不填充),如:绘制一个空心圆,则设置如下样式
    paint.setStyle(Paint.Style.STROKE);

绘制线，矩形，圆角矩形，圆形，环形，弧形等
相关api：

    //定义一个点,包含x和y坐标
    PointF point= new PointF(x, y);

    //绘制一个圆
    参数1,2:圆心坐标
    //参数3:半径
    //参数4:画笔
    canvas.drawCircle(x, y, radius, paint);

    //绘制直线
    //参数1,2:起点的x,y坐标
    //参数3,4:起点的x,y坐标
    canvas.drawLine(x1, y1,x2,y2, paint);

    //定义一个矩形
    //参数1:矩形的左边相对于屏幕左边缘的距离
    //参数2:矩形的上边相对于屏幕上边缘的距离
    //参数3:矩形的右边相对于屏幕左边缘的距离
    //参数4:矩形的下边相对于屏幕上边缘的距离
    RectF rect= new RectF(l, t, r, b);

    //绘制弧形
    //参数1:绘制弧形依赖的矩形,绘制出的弧形会与此矩形内切
    //参数2:弧形的起始角度
    //参数3:弧形的弧度
    canvas.drawArc(rect, startAngle,sweepAngle
    , false, paint);

    //绘制一个矩形
    canvas.drawRect(rect,paint);

    //绘制一个点
    //参数1,2:绘制的点的x,y坐标
    canvas.drawPoint(x,y,paint);

    //绘制多个点的x,y坐标
    float[] pts = {x1, y1, x2, y2, x3,y3...};
    //参数2:从pts集合中的哪个元素开始绘制(从哪里开始,哪里就作为x坐标,后面是y坐标,依此类推)
    //参数3:包含pts集合中共几个元素
    canvas.drawPoints(pts,offset,count,paint);
    canvas.drawRoundRect(rectF,30f,30f,paint);

    canvas的变换处理:
    Android中的坐标系:默认是以屏幕左上角为原点,x方向向右为+,y方向向下为正
    绘制界面是基于坐标系绘制的
    若调用了canvas的translate或rotate方法相当于对坐标系进行了平移和旋转,基于旋转后的坐标系进行绘制工作.
2.5．Canvas的其他处理
注:canvas在做下列处理前必须先调用save方法处理后要调用restore方法

    1,平移
    canvas.translate(100,0);
    2,旋转
    canvas.rotate(300);
    3,缩放
    canvas.scale(0.2f,0.2f);
    canvas的平移和旋转可以看成是对绘制坐标系进行的平移或旋转

#####3.自定义ViewGroup
自定义ViewGroup除了要进行上面的操作外还需要有一步，重写onLayout方法

onLayout方法就是主要为了摆放ViewGroup内部的子视图
onLayout方法参数四个参数:l,t,r,b:分别为系统赋予ViewGroup的左上右下的值  

2,常用api:

    addView():向当前控件添加子视图
    getChildCount():获取当前控件的孩子视图的个数
    getChildAt():通过指定的位置获取对应的子视图
    layout():用于对子视图的摆放处理,可以根据实际需求传入对应的l,t,r,b
For example

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
    //获取子视图总数
	    int childCount = getChildCount();
    //遍历所有的子视图
	    for (int i = 0; i < childCount; i++) {
    //获取某个子视图
		    View child = getChildAt(i);
    //将这个子视图摆放到当前空间的某个位置
    //1,横向摆放
		child.layout(30*i,0,30*(i+1),30);
    //2,纵向摆放
		//child.layout(0,30*i,30,30*(i+1));
	}
    //按照倒三角形形状摆放
	int top = 0;
	int childIndex = 0;
	for (int i = 2; i >= 0; i--) {
		for (int j = 0; j <= i; j++) {
			View child = getChildAt(childIndex);
			child.layout(30 * j, top, 30 * (j + 1), top + 30);
			childIndex++;
		}
		top += 30;
	}