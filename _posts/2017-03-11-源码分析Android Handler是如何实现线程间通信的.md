---
layout: post
title: 源码分析Android Handler是如何实现线程间通信的
categories: Android
description: Handler是如何实现线程间通信的呢？本文将从源码中分析Handler的消息通信机制
keywords: Handler,线程,Message,Looper
---

Handler作为Android消息通信的基础，它的使用是每一个开发者都必须掌握的。开发者从一开始就被告知必须在主线程中进行UI操作。但Handler是如何实现线程间通信的呢？本文将从源码中分析Handler的消息通信机制。

### 0x00 Handler使用

首先看看我们平时是如何使用的`Handler`的。先看看以下代码

```java
//定义Handler
Handler mHandler = new Handler(){
  public void handleMessage(Message msg){
    switch(msg.what){
      case UPDATE_UI:
        updateUI(msg);
        break;
    }
  }
};
class MyThread extends Thread{
  public void run(){
    //do same work!
    ...
    //send message
    Message msg = mHandler.obtainMessage(UPDATE_UI);
    mHandler.sendMessage(msg);
  }
}

private void updateUI(Message msg){
  //update UI
}
```

在子线程中`sendMessage(Message)`发送消息，然后在Handler的`handleMessage(Message)`接收消息，执行更新UI操作。那么`Handler`是如何把消息从`MyThread`传递到`MainThread`中来呢？我们从`sendMessage()`开始慢慢揭开它的面纱。

### 0x01 sendMessage(Message)

```java
public final boolean sendMessage(Message msg){
    return sendMessageDelayed(msg, 0);
}
...
public final boolean sendMessageDelayed(Message msg, long delayMillis){
    if (delayMillis < 0) {
      delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
...
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
...
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

我们发现调用`sendMessage()`方法最后都走到`enqueueMessage()`这个方法，一开始就把当前`Handler`实例赋给了`Message.target`的属性里面，后面可以知道这个`target`是用来执行处理函数回调的。

`enqueueMessage`方法是把`Message`信息放入到一个`MessageQueue`的队列中。顾名思义`MessageQueue`就是消息队列。从`sendMessageAtTime()`方法知道这个`MessageQueue`是`Handler`中的一个成员。它是在`Handler`的构造函数中通过`Loopger`对象来初始化的。

### 0x02 Handler构造函数

```java
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

这时候我们脑海知道创建`Handler`的时候，同时也创建了`Looper`实例和`MessageQueue`引用（`MessageQueue`对象其实是在`Looper`中构造的）。`Looper`是何物呢？简单地说就是消息循环，这个我们稍后会分析。

### 0x03 enqueueMessage(MessageQueue)

```java
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
          	//这里把消息插入到队列中
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```

在`MessageQueue`中可以看到这个入列方法中有一个`for`循环就是把当前的需要处理`Message`放到队列的合适位置。因为需要处理的`Message`对象都有一个开始处理的时间`when`，这个队列是按照`when`排序的。

至此，`Handler`调用`sendMessage()`方法后就把`Message`消息通过`enqueueMessage()`插入`MessageQueue`队列中。

而这个`MessageQueue`是在`Looper`中维护的。

### 0x04 prepare()创建Looper

在**0x02**中我们知道创建`Handler`时就使用静态方法`Looper.myLooper()`得到当前线程的`Looper`对象。

```java
/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```

`sThreadLocal`是一个`ThreadLocal`类型的静态变量。什么时候会把`Looper`对象放在`sThreadLocal`中呢？通过`prepare()`方法。

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

继续翻阅源码知道`Looper`在构造函数中创建`MessageQueue`对象

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}
```

调用`prepare()`方法将一个`Looper`对象放在了静态的`ThreadLocal`对象中。这个是一个与线程绑定的对象，且在内存中仅保存了一份引用。

使用`ThreadLocal`对象这一点非常巧妙，也非常重要，这是线程间通信的基础。即在线程中调用`prepare()`时就在该线程中绑定了`Looper`对象，而`Looper`对象中拥有`MessageQueue`引用。所以**每个线程都有一个消息队列**。

这样`Handler`、`Looper`、`MessageQueue`这几个类关系大概就可以画出来了。

![Handler类图](http://angrycode.qiniudn.com/Handler%E7%B1%BB%E5%9B%BE.png?imageMogr2/thumbnail/800x480/imageslim)

### 0x05 启动循环loop()

```java
/**
 * Run the message queue in this thread. Be sure to call
 * {@link #quit()} to end the loop.
 */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();
	//这里执行消息队列循环
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
      	//执行处理消息的回调
        try {
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```

`loop()`方法中有一个无限循环，不停地读取调用`MessageQueue`的`next()`方法。当`next()`没有返回时就阻塞在这里。当获取到`MessageQueue`中的消息时，就执行了处理消息的回调函数`msg.target.dispatchMessage(msg)`。

前面**0x01**分析我们知道`msg.target`是在`Handler`中的`enqueueMessage()`进行赋值，即它指向当前的`Handler`实例。

```java
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

执行`msg.target.dispatchMessage(msg)`后便走到了以下流程

```java
/**
 * Handle system messages here.
 */
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```

这里就是回调`handleMessage(msg)`函数处理消息的地方。**`Handler`负责将`Message`入列，`Looper`则负责循环从`MessageQueue`中取出需要处理的`Message`并交由Handler来处理。**

### 0x06 启动主线程的消息循环

我们知道通过静态方法`Looper.prepare()`创建了绑定当前线程的`Looper`对象，而通过`loop()`启动一个循环不停地读取队列中`Message`。但是Android系统是什么时候启动了主线程的消息循环呢？

要理解这一点就必须进入Android应用程序的入口`ActivityThread`的`main`方法。

```java
public static void main(String[] args) {
    ...

    Looper.prepareMainLooper();

    ...
    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

可以看出`main`方法中先后执行了`Looper.prepareMainLooper()`方法和`Looper.loop()`方法。正常情况下`main`方法不会退出，只有`loop()`方法发生异常后将会抛出`RuntimeException`。

### 0x07 Looper.prepareMainLooper()

```java
/**
 * Initialize the current thread as a looper, marking it as an
 * application's main looper. The main looper for your application
 * is created by the Android environment, so you should never need
 * to call this function yourself.  See also: {@link #prepare()}
 */
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}
```

`prepareMainLooper()`方法其实是调用了`prepare()`方法。

当我们启动应用时系统就调用了`prepareMainLooper()`并在主线程中绑定了一个`Looper`对象。

这时候我们回过来看看一开始的`Handler`使用方式。在主线程中我们创建了`Handler`对象，在`Handler`构造函数中初始化了`Looper`（即获取到了绑定在主线程中的`Looper`对象）。当在子线程`MyThread`中通过`mHandler.sendMessage(msg)`方法发送一个消息时就把`Message`放在与主线程绑定的`MessageQueue`中。这样在子线程中使用`Handler`就实现了消息的通信。

可以简单的使用以下类图表示，**每个线程都由一个Handler，每个Handler都是与当前所在线程的Looper绑定**。

![线程Handler模型](http://angrycode.qiniudn.com/%E7%BA%BF%E7%A8%8B%E4%B8%ADHandler%E6%A8%A1%E5%9E%8B.png)

### 0x08 主线程是否会阻塞

在**0x06**中知道在`ActivityThead`的`main`方法中启动了一个死循环。那主线程是不是就一直阻塞在这里呢？其实不然。可以看到`ActivityThread`类里面有一个自定义的`Handler`对象`mH`，在这里对象中`handleMessage()`回调中定义了`Activity`的各种交互如管理`Activity`生命周期，启动`service`，显示`window`等，都是通过`Handler`进行处理的。同时可以看出只有当应用退出`EXIT_APPLICATION`之后才回调用`Looper.quit()`停止消息循环。

```java
public void handleMessage(Message msg) {
    ...
    switch (msg.what) {
        case LAUNCH_ACTIVITY: {
            ...
            handleLaunchActivity(r, null, "LAUNCH_ACTIVITY");
            Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        } break;
        ...
        case PAUSE_ACTIVITY: {
            ...
            handlePauseActivity((IBinder) args.arg1, false,
                    (args.argi1 & USER_LEAVING) != 0, args.argi2,
                    (args.argi1 & DONT_REPORT) != 0, args.argi3);
            ...
        } break;
        
        ...
        case SHOW_WINDOW:
            ...
            handleWindowVisibility((IBinder)msg.obj, true);
            ...
            break;
        ...
        case EXIT_APPLICATION:
            if (mInitialApplication != null) {
                mInitialApplication.onTerminate();
            }
            Looper.myLooper().quit();
            break;
        ...
    }
    ...
}
```

### 0x09 总结

当创建`Handler`时将通过`ThreadLocal`在当前线程绑定一个`Looper`对象，而`Looper`持有`MessageQueue`对象。执行`Handler.sendMessage(Message)`方法将一个待处理的`Message`插入到`MessageQueue`中，这时候通过`Looper.loop()`方法获取到队列中`Message`，然后再交由`Handler.handleMessage(Message)`来处理。