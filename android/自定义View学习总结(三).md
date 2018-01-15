自定义view学习（三）
=====
这篇介绍一种自定义控件，组合控件
经常使用的界面,并且界面是由多个子控件或子布局组成,这时候需要组合控件
组合控件的作用:用于封装常用的控件组合效果,方便开发者使用
自定义组合控件的步骤：（把一个button和一个textview组合起来成为一个自定义ViewGroup）
1.创建自定义标题栏的布局文件title_bar.xml：

    <?xml version="1.0" encoding="utf-8"?>
    <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#0000ff" >

    <Button
        android:id="@+id/left_btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_margin="5dp"
        android:background="@drawable/back1_64" />

    <TextView
        android:id="@+id/title_tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="这是标题"
        android:textColor="#ffffff"
        android:textSize="20sp" />
    </RelativeLayout>
2.定义控件继承ViewGroup或ViewGroup的子类,由实际需求而定,我们这里定义相对布局

 public class TitleView extends RelativeLayout {

    // 返回按钮控件
    private Button mLeftBtn;
    // 标题Tv
    private TextView mTitleTv;

    public TitleView(Context context, AttributeSet attrs) {
        super(context, attrs);

        // 加载布局
        LayoutInflater.from(context).inflate(R.layout.title_bar, this);

        // 获取控件
        mLeftBtn = (Button) findViewById(R.id.left_btn);
        mTitleTv = (TextView) findViewById(R.id.title_tv);

    }

    // 为左侧返回按钮添加自定义点击事件
    public void setLeftButtonListener(OnClickListener listener) {
        mLeftBtn.setOnClickListener(listener);
    }

    // 设置标题的方法
    public void setTitleText(String title) {
        mTitleTv.setText(title);
    }
  }
在TitleView中主要是为自定义的标题栏加载了布局，为返回按钮添加事件监听方法，并提供了设置标题文本的方法。

3.在activity_main.xml中引入自定义的标题栏：

     <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/main_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <com.example.test.TitleView
        android:id="@+id/title_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" >
    </com.example.test.TitleView>

    </LinearLayout>

4、在MainActivity中获取自定义的标题栏，并且为返回按钮添加自定义点击事件：

    private TitleView mTitleBar;
    mTitleBar = (TitleView) findViewById(R.id.title_bar);
    mTitleBar.setLeftButtonListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "点击了返回按钮", Toast.LENGTH_SHORT)
                        .show();
                finish();
            }
        });
这样就用组合的方式实现了自定义标题栏，其实经过更多的组合还可以创建出功能更为复杂的自定义控件，比如自定义搜索栏等。

#### 总结自定义控件的绘制流程
自定义控件的基本流程为
1,加载--->构造方法
做初始化工作
2,测量--->对应onMeasure方法
用于测量视图及其子视图的大小
onSizeChanged:在onMeasure方法之后执行
3,布局--->对应onLayout方法
主要对于自定义ViewGroup来说,用于布局自己的孩子视图
requestlayout();
4,绘制--->对应onDraw方法
用于视图的绘制
Invalidate();

若需要使用系统原本不存在的属性,可使用自定义属性
onTouchEvent:MotionEvent, return true;
