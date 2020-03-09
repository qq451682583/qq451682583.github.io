---
title: Handler消息传递机制全面解析
comments: true
date: 2020-03-09 22:05:28
categories: Android源码
tags: 源码
description:
---
[TOC]
<!--more-->

###  Android的消息机制概述


Android的消息机制主要是指Handler的运行机制以及Handler所附带的MessageQueue和Looper工作过程。

handler发送Message给MessageQueue,Looper.loop循环读取MessageQueue中的msg，并调用msg.target.dispatchMessage(msg)
把消息交给handler去分发处理

handler和message是我们主动创建的，handler把message发送给MessageQueue，MessageQueue是在Looper的构造方法中初始化的，而Looper初始化的场景有2种，主线程中使用Looper可直接使用，因为应用启动的时候主线程ActivityThread的main方法中会通过Looper.prepareMainLooper()方法创建主线程的Looper，子线程使用的话必须先调用Looper.prepare()方法初始化Looper

ActivityThread通过ApplicationThread和AMS进行进程间通信，AMS以进程间通信的方式完成ActivityThread的请求后会回调ApplicationThread中的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后会将Application中的逻辑切换到ActivityThread中去执行，即切换到主线程中去执行，这个过程就是主线程的消息循环模型。

Handler创建的时候，其构造方法中会有如下代码:

```Java
''''
mLooper = Looper.myLooper();
if (mLooper == null) {
    throw new RuntimeException(
        "Can't create handler inside thread that has not called Looper.prepare()");
}
''''      

//获取Looper其实是从Looper的成员变量ThreadLocal对象sThreadLocal中获取
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
// sThreadLocal.get() will return null unless you've called prepare().
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```
可见在没有Looper的线程中是不能直接new Handler()的




整个机制所涉及到的内容：

- Handler
    
```Java
MessageQueue和Looper是它的成员变量

负责发送消息，是Message的载体 msg.target = 发送它的Handler

也可以分发消息，交给msg.callback或者自己构造函数传进来的的callback或者自己本身的handleMessage(msg)方法来处理

//构造方法
public Handler(Callback callback, boolean async)

//如何分发
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
       //mesage的callback
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            //构造方法传来的callback
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        //我们平常覆写的方法
        handleMessage(msg);
    }
}

/**
 * Subclasses must implement this to receive messages.
 */
public void handleMessage(Message msg) {
}

线程切换，Handler在任意线程发送消息，最后都会切换回Looper所在的线程执行，因为最后分发消息回调callback都在Looper的loop方法执行的
```

- Message

```
需要传递的消息，包含传递的数据，也可以设置处理消息的Callback

知识点: 创建一个一个Message 对象的时候最好使用Message.obtain()
而不是new Message();因为Message.obtain()实际是一个对象复用技术。
可以减少内存的使用。

Message中有个next对象保存下一个Message，这是Looper循环读取消息处理的原理所在

public static Message obtain() {
    //sPool 会在创建的Message使用完之后赋值
    //在recycleUnchecked()方法实现
    //赋值之前会把之前的内容清空，重用内存空间
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            sPoolSize--;
            return m;
        }
    }
    //只有第一次使用会创建新的对象
    return new Message();
}
```
- MessageQueue

```
阻塞队列(其实是单向链表实现)，因为入队出队以及延时消息会发生大
量的插入和删除操作，所以链表的数据结构效率更高，MessageQueue
中的mMessages保存链表的第一个元素

主要关注点有两个方法
enqueueMessage(Message msg, long when) 入队操作
next() 出队操作，阻塞的根本原因


首先说入队加入消息的操作：

//msg handler发送来的消息  when 延时时间
boolean enqueueMessage(Message msg, long when) {
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }
        if (msg.isInUse()) {
            throw new IllegalStateException(msg + " This message is already in use.");
        }

        synchronized (this) {
            //mQuitting 一般都为false,还有MessageQueue退出时才为true
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
                //当链表为空的时候，把第一个进来的消息赋值给为mMessages，作为链表的第一个元素
                mMessages = msg;
                //当新进来的消息是第一个那么根据mBlocked判断是否需要唤醒线程，如果不是第一个一般情况下不需要唤醒（如果加入的消息是异步的需要另外判断）
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                //是否唤醒线程的标记，只有链表的第一个消息为屏障消息，并且当前要插入的消息为异步消息并且当前线程阻塞的时候才为ture
                //只有当p.target == null也就是说当前消息没有handler载体的时候才为屏障消息
                //我们代码平常发送的消息都为同步消息
                //msg.isAsynchronous()代表是否为异步消息，当消息队列中只要有一个屏障消息后，所有的同步消息都会被屏蔽，只有异步消息会被执行通过msg.setAsynchronous(true)设置为异步消息
        
        
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                Message prev;
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

主要就是加入链表的时候按时间顺序从小到大排序，然后判断是否需要唤醒(
当没有消息时，如果需要唤醒则调用nativeWake(mPtr);来唤醒之前等待的线程

    
出队操作获取消息：

Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        //MessageQueue阻塞nextPollTimeoutMillis毫秒的时间。
//1.如果nextPollTimeoutMillis=-1，一直阻塞不会超时。
//2.如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
//3.如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)，如果期间有程序唤醒会立即返回。
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            //执行阻塞，时长nextPollTimeoutMillis
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    //如果发现了一个消息屏障，会循环找出第一个异步消息（如果有异步消息的话）， 
                    //所有同步消息都将忽略（平常发送的一般都是同步消息），可以通过setAsynchronous(boolean async)设置为异步消息。
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                
                if (msg != null) {
                    //如果有消息需要处理，先判断时间有没有到，如果没到的话设置一下阻塞时间nextPollTimeoutMillis
                    //，进入下次循环的时候会调用nativePollOnce(ptr, nextPollTimeoutMillis);阻塞；
                    //否则把消息返回给调用者，并且设置mBlocked = false代表目前没有阻塞
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    //没有消息设置-1 表示一直阻塞不会超时
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                //当没有消息或者需要延时执行时走到这 说明需要阻塞
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }

同步屏障消息：    
使用MessageQueue.postSyncBarrier()向MessageQueue中插入了一个Message，
并且未设置target。它的作用是插入一个消息屏障，
这个屏障之后的所有同步消息都不会被执行，即使时间已经到了也不会执行。
可以通过public void removeSyncBarrier(int token)来移除这个屏障，参数是post方法的返回值。
这些方法是隐藏的或者是私有的，具体应用场景可以查看ViewRootImpl中的
void scheduleTraversals()方法，它在绘图之前会插入一个消息屏障，绘制之后移除。

1.首次进入循环nextPollTimeoutMillis=0，阻塞方法
nativePollOnce(ptr,nextPollTimeoutMillis)会立即返回
2.读取列表中的消息，如果发现消息屏障，则跳过后面的同步消息，总之会通过当前时间，是否遇到屏障来返回符合条件的待处理消息
3.如果没有符合条件的消息，会处理一些不紧急的任务（IdleHandler），再次进入第一步


队列退出

 void quit(boolean safe) {
        if (!mQuitAllowed) {
            throw new IllegalStateException("Main thread not allowed to quit.");
        }

        synchronized (this) {
            if (mQuitting) {
                return;
            }
            mQuitting = true;

            if (safe) {
                removeAllFutureMessagesLocked();
            } else {
                removeAllMessagesLocked();
            }

            // We can assume mPtr != 0 because mQuitting was previously false.
            nativeWake(mPtr);
        }
    }
```

- Looper

```
Looper在消息机制中扮演者消息循环的角色，具体来说就是他会不停的从MessageQueue中查看是否有新消息，有消息会立刻处理，否则就一直阻塞在那里。

如果是在子线程中创建了Looper，那么在所有的事情完成以后应该调用quit方法来终止消息循环，负责这个字线程就会一直处于等待的状态，退出Looper后，这个线程就会立刻终止

而主线程的Looper是不可以退出的 。

//主线程的Looper
public static void prepareMainLooper() {
        //false 代表不可退出
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
//子线程的Looper 默认可退出    
public static void prepare() {
    prepare(true);
}

阻塞的问题：

Looper最重要得就是loop方法，loop方法是一个死循环，一直读取MessageQueue的消息，唯一跳出循环的
条件就是MessageQueue.next()方法返回null,从前面MessageQueue的next方法源码中我们可以看到主线程阻
塞的真正原因是因为消息队列的阻塞，当队列中没消息的时候nextPollTimeoutMillis =-1表示一直阻塞
不会超时，这时候主线程会一直循环读取MessageQueue中的消息，直到MessageQueue的quit()方法被调用，
这时候next()方法会返回null，这时候loop方法也会跳出循环，MessageQueue的quit()方法在哪被调用呢
？其实是在Looper的的quit()和quitSafely()方法中调用的。

quit():直接退出Looper
quitSafely():已有消息处理完后退出Looper退出

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

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        ''''''
    }
}

public void quit() {
    mQueue.quit(false);
}
public void quitSafely() {
    mQueue.quit(true);
}

```


## ThreadLocal

每个线程都可以有一个Looper。那么Handler如何把消息传递给正确的Looper去处理呢？

当我们new Handler()的时候，其构造方法中会从Looper的sThreadLocal对象中获取Looper对象并且通过获取的Looper对象从而获取MessageQueue,之后通过Handler发送的消息都会放到Looper中的MessageQueue中。

sThreadLocal.get()如何确保获取的Looper和Handler是在同一个线程中的呢。

这是因为sThreadLocal是一个ThreadLocal对象。

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其它线程来说则无法获取到数据。

sThreadLocal在哪存储值得呢：

```Java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }

```
在Looper.prepare()中会把创建的Looper存进去。

ThreadLocal如何实现不同线程存储对应的不同数据的呢：

存值：


```
public void set(T value) {
    //获取当前线程
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

//根据当前线程获取线程池中的ThreadLocalMap对象    
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

//首次set会创建ThreadLocalMap对象并赋值给线程的 threadLocals成员变量   
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

ThreadLocalMap是ThreadLocal内部的一个Map实现，然而它没有实现任何集合的接口规范，因为它仅供ThreadLocal内部使用，数据结构采用数组+开方地址法，内部有一个Entry类型的table数组,Entry继承WeakRefrence，是基于ThreadLocal这种特殊场景实现的Map

new ThreadLocalMap(this, firstValue); this代表当前Looper的sThreadLocal对象，值为new 出来的Looper。以Key-Value的形式存放到Entry中，并复制给ThreadLocalMap的table数组中，数组的下标的计算规则如下 :


```
private void set(ThreadLocal<?> key, Object value) {

            // We don't use a fast path as with get() because it is at
            // least as common to use set() to create new entries as
            // it is to replace existing ones, in which case, a fast
            // path would fail more often than not.

            Entry[] tab = table;
            int len = tab.length;
            //计算数组下标
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }
```

取值：


```
//ThreadLocal
public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
    
    //ThreadLocalMap
    private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

```






## 使用实例


```Java
Handler myHandler = new Handler() {
   public void handleMessage(Message msg) {
      updateUIHere();
   }
}
new Thread() {
   public void run() {
        doStuff();         // 执行耗时操作
        Message msg = myHandler.obtainMessage();
        Bundle b = new Bundle();
        b.putString("key", "value");
        m.setData(b);    // 向消息中添加数据
        myHandler.sendMessage(m);    // 向Handler发送消息，更新UI
   }
}.start();

class LooperThread extends Thread {
  *      public Handler mHandler;
  *
  *      public void run() {
  *          Looper.prepare();
  *
  *          mHandler = new Handler() {
  *              public void handleMessage(Message msg) {
  *                  // process incoming messages here
  *              }
  *          };
  *
  *          Looper.loop();
  *      }
  *  }
```

可以理解handler的作用其实是用来把在子线程执行耗时操作的结果发送到Looper所在线程去处理的类。

比如上述的实例，其实就是利用handler在子线程中更新UI

为什么不能再子线程中直接更新UI呢？


其实是因为：
1. ViewRootImp对UI操作做了验证(ViewRootImp是在OnResume启动后创建的，所以在onResume执行之前子线程其实也是可以更新UI的，相当于绕过了检查，但是不建议这么做)
    
```Java
void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }

```
2. Android的UI控件不是线程安全的，多线程并发访问可能会导致UI控件处于不可预期的状态，为什么不对UI控件访问加上锁机制呢，缺点有2个，首先加上锁机制会让UI访问的逻辑变得复杂，其次锁机制会减低UI访问的效率，应为锁机制会阻塞某些线程的执行。



为什么MessageQueue阻塞队列，不会导致ANR呢？

这是因为ANR其实是因为耗时任务执行超过一定的时间还没执行完导致的，而阻塞队列并没有执行耗时操作，只是等待消息的到来，不会阻塞线程，而且如果阻塞线程的话，后续的操作都不会执行了，但是从我们的应用启动到关闭，主线程是一直执行的，Looper并没有导致主线程不工作。

ANR超时时间的定义：
- 1.broadcast超时时间为10秒
- 按键无响应的超时时间为5秒
- 前台service无响应的超时时间为20秒
- 后台service为200秒

ANR文件的分析和获取

traces.txt系统自动生成的记录anr等异常的文件，只记录java代码产生的异常。

文件位置在/data/anr/traces.txt