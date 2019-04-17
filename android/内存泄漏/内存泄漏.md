# 内存泄漏
> hencoder 20节

## 一、GC Root 
> GC Root持有的对象不会被释放，从而造成内存泄漏。

- 激活状态的线程
- static
- Monitor Used 用于保证同步的对象，例如wait()，notify()中使用的对象、锁等。
- Stack Local 栈中的对象，每个线程都会分配一个栈，栈中的局部变量或者参数都是GC root，因为它们的引用随时可能被用到；
- JNI Local & JNI Global JNI中的局部变量和参数引用的对象；可能在JNI中定义的，也可能在虚拟机中定义

### 举例：
- AysncTask 容易造成内存泄漏的主要原因是因为，他里边有线程，而线程是一种GC Root。
    - 当在AysncTask执行中，Activity关闭了，而AysncTask内部线程是**GCroot**，并且因为MyAysncTask是MainActvity的内部类，所以MyAysncTask会持有MainActivity的对象，直到线程结束，就会出现内存泄漏。
- AysncTask 如果里边的线程没有特别长的执行时间，是不需要特别注意的。因为如果线程只是再Activity关闭之后几秒就执行完了，他就释放了。所以不用特别处理，只有当其中线程执行时间特别长的时候需要特殊处理。
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        new MyAsyncleTask().execute();
    }

    class MyAsyncleTask extends AsyncTask {

        @Override
        protected Object doInBackground(Object[] objects) {
            return null;
        }
    }
```
## 二、java内存分配策略
Java 程序运行时的内存分配策略有三种,分别是静态分配,栈式分配,和堆式分配，对应的，三种存储策略使用的内存空间主要分别是静态存储区（也称方法区）、栈区和堆区。

- 静态存储区（方法区）：主要存放静态数据、全局 static 数据和常量。这块内存在程序编译时就已经分配好，并且在程序整个运行期间都存在。
- 栈区 ：当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时这些局部变量所持有的内存将会自动被释放。因为栈内存分配运算内置于处理器的指令集中，效率很高，但是分配的内存容量有限。
- 堆区 ： 又称动态内存分配，通常就是指在程序运行时直接 new 出来的内存。这部分内存在不使用时将会由 Java 垃圾回收器来负责回收。

**栈与堆的区别：**

在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的。当在一段方法块中定义一个变量时，Java 就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给它的内存空间也将被释放掉，该内存空间可以被重新使用。

堆内存用来存放所有由 new 创建的对象（包括该对象其中的所有成员变量）和数组。在堆中分配的内存，将由 Java 垃圾回收器来自动管理。在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的引用变量。我们可以通过这个引用变量来访问堆中的对象或者数组。

## 三、常见的内存泄漏汇总
### 1、集合类泄漏
> 在这个例子中，我们循环申请Object对象，并将所申请的对象放入一个 Vector 中，如果我们仅仅释放引用本身，那么 Vector 仍然引用该对象，所以这个对象对 GC 来说是不可回收的。因此，如果对象加入到Vector 后，还必须从 Vector 中删除，

**解决方案：**   
最简单的方法就是将 Vector 对象设置为 null。

例：   
```java
    Vector v = new Vector(10);
    for (int i = 1; i < 100; i++) {
        Object o = new Object();
        v.add(o);
        o = null;    
    }
```

### 2、单例造成的内存泄漏
> 造成原因是static持有了Context对象，使得context释放不了。

**解决方案：**   
- 1、传进来context.getApplicationContext();
- 2、可以直接不传进来，直接在Application 中添加一个静态方法，然后在单例时直接MyApplication.getContext()获取；

例：
```java
    public class AppManager {
        private static AppManager instance;
        private Context context;
        private AppManager(Context context) {
            this.context = context;
        }
        public static AppManager getInstance(Context context) {
            if (instance == null) {
                instance = new AppManager(context);
            }
            return instance;
        }
    }
```
### 3、匿名内部类/非静态内部类 + 静态变量
> 非静态内部类会持有外部类的引用（所以它才可以直接访问外部类的成员变量）。上面代码中的静态handler变量间接持有了MainActivity对象。这样就造成了内存泄漏。

**解决方案：**   
解决的方法就是将内部类中对外部类的调用改成public方法，然后将Handler改成静态内部类或者外部一个类。或者将将它放到弱引用中。


例：   
```java
public class MainActivity extends AppCompatActivity {
    private static MyHandler handler = new MyHandler();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    public class MyHandler extends Handler {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);

        }
    }
}
```
### 4、Thread线程、Timer等隐藏线程
> 激活状态的线程是不会被GC回收的，所以它持有的对象也不会被回收。

>线程中持有一个Activity对象，在这个线程活跃的时间内这个Activity对象都不会被释放。

**解决方案：**   
因此，其它线程中尽量不要持有Activity,Service等大对象。如果需要用到Context，尽量使用ApplicationContext。

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AsyncTasks asyncWork = new AsyncTasks(this);
        ExecutorService defaultExecutor = Executors.newCachedThreadPool();
        defaultExecutor.execute(asyncWork);
    }

    public static class AsyncTasks implements Runnable {
        private Context context;

        public AsyncTasks(Context context) {
            this.context = context;
        }

        @Override
        public void run() {
            while (true) ;
            //正常情况下，线程执行时间不会无限，但可能有5分钟，10分钟
        }
    }
}
```
每一个Timer类都运行在一个独立的线程中。例子中我们的Timer对象的线程被设置为1000ms触发一次操作，永不结束。需要注意的是当前的引用关系Timer->TimerTask->Activity。所以当我们的Activity结束之后，还会被GC root间接持有。这个Activity每次被打开都会多一个对象在进程中，并且永远不会被回收。
解决办法就是在Activity的onDestroy方法中将Timer取消掉。


### 5、JNI Local & JNI Global

> 这类对象一般发生在参与Jni交互的类中。

比如说很多close()相关的类，InputStream,OutputStream,Cursor,SqliteDatabase等。

这些对象不止被Java代码中的引用持有，也会被虚拟机中的底层代码持有。在将持有它们的引用设置为null之前，要先将他们close()掉。

还有一个特殊的类是Bitmap。在Android系统3.0之前，它的内存一部分在虚拟机中，一部分在虚拟机外。因此它的一部分内存不参与垃圾回收，需要我们主动调用recycler()才能回收。
动态链接库中的内存是用C/C++语言申请的，这些内存不受虚拟机的管辖。所以，so库中的数组，类等都有可能发生内存泄漏，使用的时候务必小心。

### 6、注册/反注册

```java
public class AccountMananger extends Observable{
//单例的内容
}

public class MainActivity extends AppCompatActivity implements Observer {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AccountMananger.getInstance(this).addObserver(this);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
    }
    
    ...
    
    @Override
    public void update(Observable observable, Object data) {
        //todo Your logic
    }
}
```

上面的代码也会导致内存泄漏，因为注册了监听模式却没有反注册。注册过的监听者都会间接的被单例对象持有，他们都不会被GC回收。修改方法：

```java
    @Override
    protected void onDestroy() {
        super.onDestroy();
        AccountMananger.getInstance(this).deleteObserver(this);
    }
```

所有的注册型的用法都要有反注册。编码的时候养成好习惯，像Activity,Fragment等类在生命周期对等的回调方法中，最好成对的添加代码。例如在onCreate()方法注册监听之后，马上在onDestroy()方法中反注册。



## 总结：
- 1、使用静态变量的时候要小心，尤其要注意Activity/Service等大对象的传值。在单例模式中能用ApplicationContext的都用ApplicationContext，或者把聚合关系改成依赖关系，不在单例对象中持有Context引用；
- 2、养成良好的代码习惯。注册/反注册要成对出现，Activity和Service对象中避免使用非静态内部类/匿名内部类，除非十分清楚引用关系；
- 3、使用多线程的时候留意线程存活时间。尽量将聚合关系改成依赖关系，减少线程对象持有大对象的时间；
- 4、在使用xxxStream,SqlLiteDatabase,Cursor类的时候要注意释放资源。使用Timer,TimerTask的时候要记得取消任务。Bitmap在使用结束后要记得recycler()。
