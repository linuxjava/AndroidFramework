> 十年鹅厂程序员，专注大前端、个人成长、副业搞钱 <br>
> 技术交流群qiejun2025（备注：Android）

在UI线程中我们直接使用如下代码就可以使用消息系统了
```java
mMainHandler = new Handler(new Handler.Callback() {
    @Override
    public boolean handleMessage(Message message) {
        return false;
    }这个代码
});
```
和子线程使用消息系统相比，这两行重要的代码不见了
```java
Looper.prepare();
Looper.loop();
```
其实它们并非不见了，而是被系统隐藏起来了； 二者的区别：UI线程的创建是由JVM发起的，而子线程的创建是应用层开发者创建的。

当启动启动APP，首先会创建JVM虚拟机，虚拟机会加载apk文件，在这个过程中会调用ActivityThread.main（这个就是UI线程）

```java
public static void main(String[] args) {
    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    thread.attach(false, startSeq);

    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }

    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```
可见UI线程也创建了消息循环
```java
Looper.prepareMainLooper();
Looper.loop();
```
UI线程中也创建是属于自己的Looper和MessageQueue，并且使用static变量sMainLooper保存
```java
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
这样我们可以通过如下方法快速获取到UI线程的Looper对象

```java
Looper.getMainLooper()
```
当调用Looper.loop()后，UI线程会一直阻塞直到有消息过来将其唤醒，这也是Android系统被称为事件驱动的原因。





