> 十年鹅厂程序员，专注大前端、个人成长、副业搞钱 <br>
> 技术交流群qiejun2025（备注：Android）

消息系统在线程通过的过程中可能会创建大量的Message，那么就需要JVM不断创建和回收这些对象，进而影响系统的性能，因此才有了消息回收机制。
![类图.jpg](https://raw.githubusercontent.com/linuxjava/AndroidFramework/refs/heads/main/01%20Handler%E6%B6%88%E6%81%AF%E7%B3%BB%E7%BB%9F/images/%E7%B1%BB%E5%9B%BE.jpg)
Message中的static成员sPool和sPoolSize便是消息回收机制的回收队列，它会将分发处理完的消息添加到队列中，当下次需要创建Message时，如果队列中存在空闲的消息则直接复用，无需系统再创建。

# 消息回收

在Looper.loop方法中，当消息被分发处理完后就会对其进行回收

```java
public static void loop() {
    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

       
        try {
            //消息分发处理
            msg.target.dispatchMessage(msg);
        } catch (Exception exception) {
            if (observer != null) {
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }
        //消息回收
        msg.recycleUnchecked();
    }
}
```

再来看看Meesage类中recycleUnchecked方法

```java
void recycleUnchecked() {
    synchronized (sPoolSync) {
        if (sPoolSize < MAX_POOL_SIZE) {
            next = sPool;
            sPool = this;
            sPoolSize++;
        }
    }
}
```

这里直接将回收的消息插入到sPool链表头部，并sPoolSize++，消息回收队里最大只允许有MAX\_POOL\_SIZE=50 个空闲的消息对象。

除了在Looper.loop方法中回收消息外，还在如下几个地方也会回收消息

*   调用Looper.quit和Looper.quitSafely消息系统退出时
*   调用Handler.removeMessages和removeCallbacks

# 消息复用

调用Message.obtain相关方法时会复用sPool中闲置的消息

```java
public static Message obtain() {
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
    return new Message();
}
```
