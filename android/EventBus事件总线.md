<!--
author: 冷火-王胜 
date: 2017-2-17 
title: EventBus事件总线
tags: Android Message
category: Android
status: publish 
-->
什么是EventBus:
EventBus是android下的高效的发布/订阅总线机制,它的作用是可以代替传统的Intent,Handler,Broadcast,或者接口函数在Fragment,Activity,Service,线程之间传递数据,执行方法,它用来传递数据特别方便,并且代码简洁,实现了代码之间的解耦.是一种发布订阅者模式,或者称为观察者设计模式
来一张EventBus的github上的框架原理流程图

![图片1.png](http://upload-images.jianshu.io/upload_images/4669002-909f0e394bdab20c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

怎么使用EventBus:
1.引入EventBus,这里有两种方法,一种是下载EventBus库源码地址：https://github.com/greenrobot/EventBus, 库地址：https://github.com/greenrobot/EventBus/releases, ,将EventBus的jar文件放入libs中,第二中方法是直接在build.gradle中添加依赖, compile 'org.greenrobot:eventbus:3.0.0',

2.定义事件,前面已经说了,EventBus是事件发布总线,用来传递消息的,那么我们需要定义要传递的消息.定义一个类,继承默认的Object即可,用来区分事件和传递数据,

3.添加订阅者,一个类用成为订阅者类,就必须注册, EventBus.getDefault().register(this);通过注册成为订阅者之后,框架会通过反射机制获取所有的方法和参数.
那么订阅者是怎么接收发布者发布的事件的,这里就要讲一下不同版本的EventBus的订阅者接收事件的方法不一样了,EventBus2.4的时候订阅者所在的类可以定义以下一个或多个方法用以接收事件:
 
        public void onEvent(MsgEvent1 msg)// 与发布者在同一个线程
        public void onEventMainThread(MsgEvent1 msg)// 执行在主线程。
        public void onEventBackgroundThread(MsgEvent1 msg)// 执行在子线程，如果发布者是子线程则直接执行，如果发布者不是子线程，则创建一个再执行
        public void onEventAsync(MsgEvent1 msg)// 执行在在一个新的子线程
而在EventBus3.0之后就取消了约定好的方法定义,并提供了注解的方式进行监听. 

         @Subscribe(threadMode = ThreadMode.MAIN)
         public void getEventBus(MyEvent event){
         }
threadMode就是旧版本接收信息运行的方法。

4.发布者发布事件,所有类都可以成为发布者,订阅者也可以作为发布者给自己发布消息, EventBus.getDefault().post(new MsgEvent1("主线程发的消息1"));一旦执行了此方法， 所有订阅者都会执行第二步定义的方法。

5.取消订阅,一般订阅者销毁的时候在销毁的生命周期方法中可以取消订阅,不再接收消息, EventBus.getDefault().unregister(this)
需要注意的是发布者可以发布任何事件, 订阅者接受消息时，只要定义的是第二步四个方法任意一个，并且参数和发布者发布的一致，即可被执行。发布者也可以通过第二步接收消息，订阅者也可以作为发布者发消息给自己。

下面通过一个Demo更好的认识EventBus:

引入EventBus,这里我用的是gradle的方式:  

      compile 'org.greenrobot:eventbus:3.0.0'
主界面: 

       /**因为主界面中要实现左边一个fragment,右边一个fragment,所以继承FragmentActivity*/
    public class MainActivity extends FragmentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
    }
主界面的xml布局文件 

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:divider="?android:attr/dividerHorizontal"
              android:orientation="horizontal"
              android:showDividers="middle"
              android:baselineAligned="false"
              tools:context=".MainActivity" >

    <fragment
        android:id="@+id/left_fragment"
        android:name="com.xinguangnet.eventdemo.LeftFragment"
        android:layout_width="0dip"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/right_fragment"
        android:name="com.xinguangnet.eventdemo.RightFragment"
        android:layout_width="0dip"
        android:layout_height="match_parent"
        android:layout_weight="3" />

    </LinearLayout>

定义事件: 

    public class MsgEvent1 {
    private String msg;

    public MsgEvent1(String msg) {
        super();
        this.msg = msg;
    }
    public String getMsg() {
        return msg;
    }
    }
右边的fragment,作为接收者:
创建的时候注册,销毁的时候取消注册: 

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //界面创建时就订阅事件接收消息
    EventBus.getDefault().register(this);
    }

    @Override
    public void onDestroy() {
    super.onDestroy();
    //界面销毁时取消订阅
    EventBus.getDefault().unregister(this);
    }
接收事件的方法: 

    /**
     * 与发布者在同一个线程
     * @param msg 事件1
     */
    @Subscribe
    public void onEvent(MsgEvent1 msg){
    String content = msg.getMsg()
            + "\n ThreadName: " + Thread.currentThread().getName()
            + "\n ThreadId: " + Thread.currentThread().getId();
    System.out.println("onEvent(MsgEvent1 msg)收到" + content);
    }

    /**
     * 执行在主线程。
     * 非常实用，可以在这里将子线程加载到的数据直接设置到界面中。
     * @param msg 事件1
     */
    @Subscribe
    public void onEventMainThread(MsgEvent1 msg){
        String content = msg.getMsg()
                + "\n ThreadName: " + Thread.currentThread().getName()
                  + "\n ThreadId: " + Thread.currentThread().getId();
        System.out.println("onEventMainThread(MsgEvent1 msg)收到" + content);
        tv.setText(content);
    }

    /**
     * 执行在子线程，如果发布者是子线程则直接执行，如果发布者不是子线程，则创建一个再执行
     * 此处可能会有线程阻塞问题。
     * @param msg 事件1
     */
    @Subscribe
    public void onEventBackgroundThread(MsgEvent1 msg){
        String content = msg.getMsg()
                + "\n ThreadName: " + Thread.currentThread().getName()
                + "\n ThreadId: " + Thread.currentThread().getId();
        System.out.println("onEventBackgroundThread(MsgEvent1 msg)收到" + content);
    }

    /**
     * 执行在在一个新的子线程
     * 适用于多个线程任务处理， 内部有线程池管理。
     * @param msg 事件1
     */
    @Subscribe
    public void onEventAsync(MsgEvent1 msg){
        String content = msg.getMsg()
                + "\n ThreadName: " + Thread.currentThread().getName()
                + "\n ThreadId: " + Thread.currentThread().getId();
        System.out.println("onEventAsync(MsgEvent1 msg)收到" + content);
    }

    /**
     * 与发布者在同一个线程
     * @param msg 事件2
     */
    @Subscribe
    public void onEvent(MsgEvent2 msg){
        String content = msg.getMsg()
            + "\n ThreadName: " + Thread.currentThread().getName()
            + "\n ThreadId: " + Thread.currentThread().getId();
    System.out.println("onEvent(MsgEvent2 msg)收到" + content);
    tv.setText(content);
    }

左边的fragment用来发布消息: 

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    String[] strs = new String[]{"主线程消息1", "子线程消息1", "主线程消息2"};
    setListAdapter(new ArrayAdapter<String>(getActivity(), android.R.layout.simple_list_item_1, strs));
    }

    @Override
    public void onListItemClick(ListView l, View v, int position, long id) {
    switch (position) {
        case 0:
            // 主线程
            System.out.println(
                    "----------------------主线程发的消息1"
                            + " threadName: " + Thread.currentThread().getName()
                            + " threadId: " + Thread.currentThread().getId());
            EventBus.getDefault().post(new MsgEvent1("主线程发的消息1"));
            break;
        case 1:
            // 子线程
            new Thread() {
                public void run() {
                    System.out.println(
                            "----------------------子线程发的消息1"
                                    + " threadName: " + Thread.currentThread().getName()
                                    + " threadId: " + Thread.currentThread().getId());
                    EventBus.getDefault().post(new MsgEvent1("子线程发的消息1"));
                }

                ;
            }.start();

            break;
        case 2:
            // 主线程
            System.out.println(
                    "----------------------主线程发的消息2"
                            + " threadName: " + Thread.currentThread().getName()
                            + " threadId: " + Thread.currentThread().getId());
            EventBus.getDefault().post(new MsgEvent2("主线程发的消息2"));
            break;
    }
    }
这个Demo写完之后运行起来有一个出现了bug,我开始百思不得其解,后来反复看了错误日志,终于知道了原因


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-2ee6adac224e9e66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我引入的EventBus3.0但是一开始在订阅者的接收方法里面没有加注解,所以报了以上的错误
