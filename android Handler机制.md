## android Handler机制
#### 消息驱动四要素
* 接收消息的“消息队列”                                                            MessageQueue
* 阻塞式地从消息队列中接收消息并进行处理的“线程”             Thread+Looper
* 可发送的“消息的格式”                                                            Message
* “消息发送函数”                                                                       Handler的post和sendMessage

> Looper是一个循环器，不断从MessageQueue中取Message或者Runnable，而Handler可以看做是一个Looper的暴露接口，向外部暴露一些事件，并暴露sendMessage()和post()函数。

#### Handler的创建
我们要先调用Looper.prepare()方法，才能创建Handler对象（主线程中可以直接创建Handler对象，而在子线程中需要先调用Looper.prepare()才能创建Handler对象。）

因为Looper.myLooper()会从当前线程中取出Looper对象，如果当前线程中没有looper对象就返回null，如果返回null，Looper.prepare()就会抛出异常。主线程已经帮我们自动调用了Looper.prepare()方法。

#### Handler发送消息
handler.sendMessage()        然后在Handler的handleMessage()方法中重新得到这条Message

其中除了sendMessageAtFrontOfQueue()方法之外，其它的发送消息方法最终都会辗转调用到sendMessageAtTime()方法中 

    public boolean sendMessageAtTime(Message msg, long uptimeMillis)  
    {  
    boolean sent = false;  
    MessageQueue queue = mQueue;  
    if (queue != null) {  
        msg.target = this;  
        sent = queue.enqueueMessage(msg, uptimeMillis);  
    }  
    else {  
        RuntimeException e = new RuntimeException(  
            this + " sendMessageAtTime() called with no mQueue");  
        Log.w("Looper", e.getMessage(), e);  
    }  
    return sent;  
    }  

sendMessageAtTime()方法接收两个参数，其中msg参数就是我们发送的Message对象，而uptimeMillis参数则表示发送消息的时间，它的值等于自系统开机到当前时间的毫秒数再加上延迟时间，如果你调用的不是sendMessageDelayed()方法，延迟时间就为0，然后将这两个参数都传递到MessageQueue的enqueueMessage()方法中。enqueueMessage()是一个入队方法

    final boolean enqueueMessage(Message msg, long when) {  
    if (msg.when != 0) {  
        throw new AndroidRuntimeException(msg + " This message is already in use.");  
    }  
    if (msg.target == null && !mQuitAllowed) {  
        throw new RuntimeException("Main thread not allowed to quit");  
    }  
    synchronized (this) {  
        if (mQuiting) {  
            RuntimeException e = new RuntimeException(msg.target + " sending message to a Handler on a dead thread");  
            Log.w("MessageQueue", e.getMessage(), e);  
            return false;  
        } else if (msg.target == null) {  
            mQuiting = true;  
        }  
        msg.when = when;  
        Message p = mMessages;  
        if (p == null || when == 0 || when < p.when) {  
            msg.next = p;  
            mMessages = msg;  
            this.notify();  
        } else {  
            Message prev = null;  
            while (p != null && p.when <= when) {  
                prev = p;  
                p = p.next;  
            }  
            msg.next = prev.next;  
            prev.next = msg;  
            this.notify();  
        }  
    }  
    return true;  
    }  

MessageQueue是一个消息队列，用于将所有收到的消息以队列的形式进行排列，并提供入队和出队的方法。这个类是在Looper的构造函数中创建的，因此一个Looper也就对应了一个MessageQueue。

#### handleMessage()接收消息（Looper.loop()）
有了入队操作，出队操作是不断地调用的MessageQueue的next()方法，这个next()方法就是消息队列的出队方法。每当有一个消息出队，就将它传递到msg.target的dispatchMessage()方法中，那这里msg.target又是什么呢？其实就是Handler，Handler中dispatchMessage()方法中有handleMessage(msg); 所以最终都会走到handle Message（）方法里。

    public static final void loop() {  
    Looper me = myLooper();  
    MessageQueue queue = me.mQueue;  
    while (true) {  
        Message msg = queue.next(); // might block  
        if (msg != null) {  
            if (msg.target == null) {  
                return;  
            }  
            if (me.mLogging!= null) me.mLogging.println(  
                    ">>>>> Dispatching to " + msg.target + " "  
                    + msg.callback + ": " + msg.what  
                    );  
            msg.target.dispatchMessage(msg);  
            if (me.mLogging!= null) me.mLogging.println(  
                    "<<<<< Finished to    " + msg.target + " "  
                    + msg.callback);  
            msg.recycle();  
        }  
    }  
    }  


#### 为什么使用异步消息处理的方式就可以对UI进行操作了呢？

这是由于Handler总是依附于创建时所在的线程，比如我们的Handler是在主线程中创建的，而在子线程中又无法直接对UI进行操作，于是我们就通过一系列的发送消息、入队、出队等环节，最后调用到了Handler的handleMessage()方法中，这时的handleMessage()方法已经是在主线程中运行的，因而我们当然可以在这里进行UI操作了。