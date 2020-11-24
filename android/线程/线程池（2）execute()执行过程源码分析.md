
# 线程池ThreadPoolExecutor源码分析
### 所需知识点：
- 1、ReentranLock 重入锁 以及 Condition的联合使用。
- 不可重入的互斥锁，AQS AbstractQueuedSynchronizer 
- AQS是怎么实现不可重入互斥锁的。
- 2、AtomicInteger 线程安全的int
- 3、volatile 线程安全关键字
- 4、BlockingQueue 阻塞线程队列


# 源码分析execute()执行过程
老规矩我们先上图：

![image](https://note.youdao.com/yws/public/resource/49dd430837728c8137d8c1d59164db4e/xmlnote/WEBRESOURCEaaf95e78502911bbb581ce6118683526/6236)
> 1、如果线程池中的线程数量少于corePoolSize，就创建新的线程来执行新添加的任务
> 
> 2、如果线程池中的线程数量大于等于corePoolSize，但队列workQueue未满，则将新添加的任务放到workQueue中
> 
> 3、如果线程池中的线程数量大于等于corePoolSize，且队列workQueue已满，但线程池中的线程数量小于maximumPoolSize，则会创建新的线程来处理被添加的任务
> 
> 4、如果线程池中的线程数量等于了maximumPoolSize，就用RejectedExecutionHandler来执行拒绝策略


- [一、ThreadPoolExecutor.execute()方法](#jump1)
- [二、ThreadPoolExecutor.addWorker()方法](#jump2)
- [三、Worker内部类](#jump3)
- [四、ThreadPoolExecutor.runWorker()](#jump4)
- [五、ThreadPoolExecutor.getTask()方法](#jump5)
- [六、ThreadPoolExecutor.processWorkerExit()方法](#jump6)
- [七、ThreadPoolExecutor.tryTerminate()方法](#jump7)
- [八、ThreadPoolExecutor.interruptIdleWorkers()方法](#jump8)

## <span id="jump1">一、ThreadPoolExecutor.execute()方法</span>
流程图：
![image](https://note.youdao.com/yws/public/resource/49dd430837728c8137d8c1d59164db4e/xmlnote/WEBRESOURCE86d92c865bca12f085e007dcf30b649f/6243)

```java
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 如果运行的线程少于corePoolSize，尝试开启一个新线程去运行command，command作为这个线程的第一个任务
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     * 如果任务成功放入队列，我们仍需要一个双重校验去确认是否应该新建一个线程（因为可能存在有些线程在我们上次检查后死了） 或者 从我们进入这个方法后，pool被关闭了
     * 所以我们需要再次检查state，如果线程池停止了需要回滚入队列，如果池中没有线程了，新开启 一个线程
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     * 如果无法将任务入队列（可能队列满了），需要新开区一个线程（自己：往maxPoolSize发展）
     * 如果失败了，说明线程池shutdown 或者 饱和了，所以我们拒绝任务
     */
    //获取线程池当前状态
    int c = ctl.get();
    /**
     * 1、获取当前工作的Worker数量，如果小于核心线程池数量。就创建新的Worker
     */
    if (workerCountOf(c) < corePoolSize) {
        //创建新worker对象，启动新线程，并且设置为true 核心线程。
        //如果添加创建成功直接返回。
        if (addWorker(command, true))
            return;
        //新增Worker失败，重新获取线程池状态值
        /**
         * 没有成功addWorker()，再次获取c（凡是需要再次用ctl做判断时，都会再次调用ctl.get()）
         * 失败的原因可能是：
         * 1、线程池已经shutdown，shutdown的线程池不再接收新任务
         * 2、workerCountOf(c) < corePoolSize 判断后，由于并发，别的线程先创建了worker线程，导致workerCount>=corePoolSize
         */
        c = ctl.get();
    }


    /**
     * 2、走到这，说明核心线程池已满，或者线程池shutdown了。
     * 如果线程池在运行状态，并且workQueue队列插入成功。
     * BlockQueue #offer()特性，插入值失败返回false
     */
    if (isRunning(c) && workQueue.offer(command)) {
        //再次校验位
        int recheck = ctl.get();
        /**
         * 再次校验放入workerQueue中的任务是否能被执行
         * 1、如果线程池不是运行状态了，应该拒绝添加新任务，从workQueue中删除任务
         * 2、如果线程池是运行状态，或者从workQueue中删除任务失败（刚好有一个线程执行完毕，并消耗了这个任务），
         *  那么addWorker(null,false)确保还有线程执行任务（只要有一个就够了）
         */

        //如果线程池不再运行状态，并且workQueue成功删除了刚添加的任务，那么就调用拒绝handler方法。
        if (! isRunning(recheck) && remove(command))
            reject(command);
        //如果当前worker数量为0，通过addWorker(null, false)创建一个线程，其任务为null
        //为什么只检查运行的worker数量是不是0呢？？ 为什么不和corePoolSize比较呢？？
        //只保证有一个worker线程可以从queue中获取任务执行就行了？？
        //因为只要还有活动的worker线程，就可以消费workerQueue中的任务
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);//第一个参数为null，说明只为新建一个worker线程，没有指定firstTask
        //第二个参数为true代表占用corePoolSize，false占用maxPoolSize
    }


    /**
     * 3、如果添加workQueue队列失败，那么就尝试添加非核心线程。
     * 直到线程扩容超过maximumPoolSize，addWorker失败就会调用拒绝handler方法
     */
    else if (!addWorker(command, false))
        reject(command);
}
```

总结分析：
execute(Runnable command)

参数：

    command    提交执行的任务，不能为空
    
执行流程：

1、如果线程池当前线程数量少于corePoolSize，则addWorker(command, true)创建新worker线程，如创建成功返回，如没创建成功，则执行后续步骤；

    addWorker(command, true)失败的原因可能是：
    A、线程池已经shutdown，shutdown的线程池不再接收新任务
    B、workerCountOf(c) < corePoolSize 判断后，由于并发，别的线程先创建了worker线程，导致workerCount>=corePoolSize
    
2、如果线程池还在running状态，将task加入workQueue阻塞队列中，如果加入成功，进行double-check，如果加入失败（可能是队列已满），则执行后续步骤；

    double-check主要目的是判断刚加入workQueue阻塞队列的task是否能被执行
    A、如果线程池已经不是running状态了，应该拒绝添加新任务，从workQueue中删除任务
    B、如果线程池是运行状态，或者从workQueue中删除任务失败（刚好有一个线程执行完毕，并消耗了这个任务），确保还有线程执行任务（只要有一个就够了）
    
3、如果线程池不是running状态 或者 无法入队列，尝试开启新线程，扩容至maxPoolSize，如果addWork(command, false)失败了，拒绝当前command


## <span id="jump2">二、ThreadPoolExecutor.addWorker()方法</span>
![image](https://note.youdao.com/yws/public/resource/49dd430837728c8137d8c1d59164db4e/xmlnote/WEBRESOURCE9d57398b731443fd33614e49e86b20c9/6251)

```java
  /**
 * Checks if a new worker can be added with respect to current
 * pool state and the given bound (either core or maximum). If so,
 * the worker count is adjusted accordingly, and, if possible, a
 * new worker is created and started, running firstTask as its
 * first task. This method returns false if the pool is stopped or
 * eligible to shut down. It also returns false if the thread
 * factory fails to create a thread when asked.  If the thread
 * creation fails, either due to the thread factory returning
 * null, or due to an exception (typically OutOfMemoryError in
 * Thread.start()), we roll back cleanly.
 *
 * @param firstTask the task the new thread should run first (or
 * null if none). Workers are created with an initial first task
 * (in method execute()) to bypass queuing when there are fewer
 * than corePoolSize threads (in which case we always start one),
 * or when the queue is full (in which case we must bypass queue).
 * Initially idle threads are usually created via
 * prestartCoreThread or to replace other dying workers.
 *
 * @param core if true use corePoolSize as bound, else
 * maximumPoolSize. (A boolean indicator is used here rather than a
 * value to ensure reads of fresh values after checking other pool
 * state).
 * @return true if successful
 *  检查根据当前线程池的状态和给定的边界(core or maximum)是否可以创建一个新的worker
 *  * 如果是这样的话，worker的数量做相应的调整，如果可能的话，创建一个新的worker并启动，参数中的firstTask作为worker的第一个任务
 *  * 如果方法返回false，可能因为pool已经关闭或者调用过了shutdown
 *  * 如果线程工厂创建线程失败，也会失败，返回false
 *  * 如果线程创建失败，要么是因为线程工厂返回null，要么是发生了OutOfMemoryError
 */
private boolean addWorker(Runnable firstTask, boolean core) {
    //外层循环，负责判断线程池状态
    retry:
    for (;;) {
        int c = ctl.get();
        //线程池状态
        int rs = runStateOf(c);

        // Check if queue empty only if necessary.
        /**
         * 线程池的state越小越是运行状态，running=-1，shutdown=0, stop=1, tidying=2，terminated=3
         * 1、如果线程池state已经至少是shutdown状态了
         * 2、并且以下3个条件任意一个是false
         *   rs == SHUTDOWN         （隐含：rs>=SHUTDOWN）false情况： 线程池状态已经超过shutdown，可能是stop、tidying、terminated其中一个，即线程池已经终止
         *   firstTask == null      （隐含：rs==SHUTDOWN）false情况： firstTask不为空，rs==SHUTDOWN 且 firstTask不为空，return false，场景是在线程池已经shutdown后，还要添加新的任务，拒绝
         *   ! workQueue.isEmpty()  （隐含：rs==SHUTDOWN，firstTask==null）false情况： workQueue为空，当firstTask为空时是为了创建一个没有任务的线程，再从workQueue中获取任务，如果workQueue已经为空，那么就没有添加新worker线程的必要了
         * return false，即无法addWorker()
         */
        if (rs >= SHUTDOWN &&
                ! (rs == SHUTDOWN && firstTask == null && !workQueue.isEmpty())
                )
            return false;

        //内层循环，负责worker数量+1
        for (;;) {
            //获取当前worker数量
            int wc = workerCountOf(c);
            //如果worker数量>线程池最大上限CAPACITY（即使用int低29位可以容纳的最大值）
            //或者( worker数量>corePoolSize 或  worker数量>maximumPoolSize )，即已经超过了给定的边界
            if (wc >= CAPACITY ||
                    wc >= (core ? corePoolSize : maximumPoolSize))
                return false;

            //调用unsafe CAS操作，使得worker数量+1，成功则跳出外层retry循环
            //CAS 即Compare and Swap 调用AtomicInteger的同步+1方法，这个方法可能会失败，返回true成功，false失败
            if (compareAndIncrementWorkerCount(c))
                break retry;

            //重新验证线程池运行状态
            c = ctl.get();  // Re-read ctl
            //如果当前状态和外层循环开始时不一样了，那么回到外层循环重新开始。
            if (runStateOf(c) != rs)
                continue retry;
            // else CAS failed due to workerCount change; retry inner loop
        }
    }


    /**
     * worker数量+1成功的后续操作
     * 添加到workers Set集合，并启动worker线程
     */
    boolean workerStarted = false;
    boolean workerAdded = false;
    Worker w = null;
    try {
        //1、设置worker这个AQS锁的同步状态state=-1
        //2、将firstTask设置给worker的成员变量firstTask
        //3、使用worker自身这个runnable，调用ThreadFactory创建一个线程，并设置给worker的成员变量thread
        w = new Worker(firstTask);
        final Thread t = w.thread;
        if (t != null) {
            final ReentrantLock mainLock = this.mainLock;
            mainLock.lock();
            try {
                //--------------------------------------------这部分代码是上锁的
                // Recheck while holding lock.
                // Back out on ThreadFactory failure or if
                // shut down before lock acquired.
                // 当获取到锁后，再次检查线程池运行状态，
                int rs = runStateOf(ctl.get());

                //如果线程池在运行running<shutdown 或者 线程池已经shutdown，且firstTask==null（可能是workQueue中仍有未执行完成的任务，创建没有初始任务的worker线程执行）
                //worker数量-1的操作在addWorkerFailed()
                if (rs < SHUTDOWN ||
                        (rs == SHUTDOWN && firstTask == null)) {
                    if (t.isAlive()) // precheck that t is startable 线程已经启动，抛非法线程状态异常
                        throw new IllegalThreadStateException();

                    //workers是一个HashSet<Worker>,将worker存入
                    workers.add(w);
                    int s = workers.size();
                    if (s > largestPoolSize)
                        largestPoolSize = s;

                    //标识worker添加成功
                    workerAdded = true;
                }
            } finally {
                mainLock.unlock();
            }
            //添加成功，启动线程
            if (workerAdded) {
                t.start();
                workerStarted = true;
            }
        }
    } finally {
        //如果启动失败，回滚操作
        if (! workerStarted)
            addWorkerFailed(w);
    }
    return workerStarted;
}

/**
 * 如果worker不为空，就从workQueue移除
 * @param w
 */
private void addWorkerFailed(Worker w) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        if (w != null)
            workers.remove(w);
        //里边就是调用AtomicInteger#compareAndSet(expect, expect - 1) 减了1
        decrementWorkerCount();
        tryTerminate();
    } finally {
        mainLock.unlock();
    }
}

```
addWorker(Runnable firstTask, boolean core)

参数：

    firstTask：    worker线程的初始任务，可以为空
    core：           true：将corePoolSize作为上限，false：将maximumPoolSize作为上限
addWorker方法有4种传参的方式：

    1、addWorker(command, true)

    2、addWorker(command, false)

    3、addWorker(null, false)

    4、addWorker(null, true)

在execute方法中就使用了前3种，结合这个核心方法进行以下分析

    第一个：线程数小于corePoolSize时，放一个需要处理的task进Workers Set。如果Workers Set长度超过corePoolSize，就返回false
    第二个：当队列被放满时，就尝试将这个新来的task直接放入Workers Set，而此时Workers Set的长度限制是maximumPoolSize。如果线程池也满了的话就返回false
    第三个：放入一个空的task进workers Set，长度限制是maximumPoolSize。这样一个task为空的worker在线程执行的时候会去任务队列里拿任务，这样就相当于创建了一个新的线程，只是没有马上分配任务
    第四个：这个方法就是放一个null的task进Workers Set，而且是在小于corePoolSize时，如果此时Set中的数量已经达到corePoolSize那就返回false，什么也不干。实际使用中是在prestartAllCoreThreads()方法，这个方法用来为线程池预先启动corePoolSize个worker等待从workQueue中获取任务执行
执行流程：

    1、判断线程池当前是否为可以添加worker线程的状态，可以则继续下一步，不可以return false：
        A、线程池状态>shutdown，可能为stop、tidying、terminated，不能添加worker线程
        B、线程池状态==shutdown，firstTask不为空，不能添加worker线程，因为shutdown状态的线程池不接收新任务
        C、线程池状态==shutdown，firstTask==null，workQueue为空，不能添加worker线程，因为firstTask为空是为了
        添加一个没有任务的线程再从workQueue获取task，而workQueue为空，说明添加无任务线程已经没有意义
    2、线程池当前线程数量是否超过上限（corePoolSize 或 maximumPoolSize），超过了return false，没超过则对workerCount+1，继续下一步
    3、在线程池的ReentrantLock保证下，向Workers Set中添加新创建的worker实例，添加完成后解锁，并启动worker线程，
    如果这一切都成功了，return true，如果添加worker入Set失败或启动失败，调用addWorkerFailed()逻辑

## <span id="jump3">三、Worker内部类</span>
```java
 /**
 * Class Worker mainly maintains interrupt control state for
 * threads running tasks, along with other minor bookkeeping.
 * This class opportunistically extends AbstractQueuedSynchronizer
 * to simplify acquiring and releasing a lock surrounding each
 * task execution.This protects against interrupts that are
 * intended to wake up a worker thread waiting for a task from
 * instead interrupting a task being run.
 *
 * We implement a simple non-reentrant mutual exclusion lock rather than use
 * ReentrantLock because we do not want worker tasks to be able to
 * reacquire the lock when they invoke pool control methods like
 * setCorePoolSize.
 *
 * Additionally, to suppress interrupts until
 * the thread actually starts running tasks, we initialize lock
 * state to a negative value, and clear it upon start (in
 * runWorker).
 *
 * Worker类大体上管理着运行线程的中断状态 和 一些指标
 * Worker类投机取巧的继承了AbstractQueuedSynchronizer来简化在执行任务时的获取、释放锁
 * 这样防止了中断在运行中的任务，只会唤醒(中断)在等待从workQueue中获取任务的线程
 *   解释：
 *    为什么不直接在execute(runnable command)直接执行command，而要包一层Worker呢？
 *        1、主要目的是为了控制线程中断，使用不可重入的互斥锁AQS，来限制同一线程中其他操作导致线程中断。
 *        2、正常shutdown()方法，调用的是interruptIdleWorkers()，这个方法是需要w.tryLock()的，
 *        也就是用不可重入锁AQS协助拦截正在运行的线程调用t.intercept()中断。所以shutdown()方法
 *        不会中断正在执行任务的worker线程。
 *        3、但如果shutdownNow()方法，调用的是interruptWorkers()，这个方法并不加锁，而是直接遍历
 *        所有worker，并t.intercept()。所以shutdownNow()相当于会中断正在执行的Worker线程。
 *
 * worker实现了一个简单的不可重入的互斥锁，而不是用ReentrantLock可重入锁
 * 因为我们不想让在调用比如setCorePoolSize()这种线程池控制方法时可以再次获取锁(重入)
 *   解释：
 *     1、setCorePoolSize()时会调用interruptIdleWorkers()，通过这个方法里w.tryLock()来拦截
 *     利用不可重入锁的特性，保证同一线程中执行时也会阻塞，来保证worker不被中断。
 *     2、类似的方法还有（只要调用interruptIdleWorkers()的全是）：
 *     shutdown()
 *     setMaximumPoolSize()
 *     setKeepAliveTime()
 *     allowCoreThreadTimeOut()
 *
 * 另外，为了保证只有worker中的线程已经在运行状态才能被中断。我们初始化state = -1，并在runWorker()
 * 启动线程时，将state设置 = 0。
 *    解释：
 *      1、创建Worker过程并没有真正的t.start()启动线程。在runWorker()中才调用启动线程。
 *      2、所以在t.start()之前并没有必要去中断线程。只有state >= 0的时候，才表示有线程可中断。
 */
private final class Worker
        extends AbstractQueuedSynchronizer
        implements Runnable
{
    /**
     * This class will never be serialized, but we provide a
     * serialVersionUID to suppress a javac warning.
     *
     * 这个类永远都不会被序列化，我们提供序列号id只是解决javac 的警告
     */
    private static final long serialVersionUID = 6138294804551838833L;

    /** Thread this worker is running in.  Null if factory fails. */
    final Thread thread;
    /** Initial task to run.  Possibly null. */
    Runnable firstTask;
    /** Per-thread task counter 记录已经完成的任务数*/
    volatile long completedTasks;

    /**
     * Creates with given first task and thread from ThreadFactory.
     * @param firstTask the first task (null if none)
     */
    Worker(Runnable firstTask) {
        //初始化state为-1，在runWorker()调用t.start()时再设置为0
        setState(-1); // inhibit interrupts until runWorker
        //任务可能为null，为空时runWorker()时就会自旋，getTask()，不断获取workQueue中的任务。
        this.firstTask = firstTask;
        //通过线程工厂创建线程
        this.thread = getThreadFactory().newThread(this);
    }

    /** Delegates main run loop to outer runWorker  */
    public void run() {
        runWorker(this);
    }

    // Lock methods
    //
    // The value 0 represents the unlocked state.
    // The value 1 represents the locked state.
    //
    // 获取锁状态，0表示未锁，1表示已锁

    protected boolean isHeldExclusively() {
        return getState() != 0;
    }

    /**
     * 尝试获取锁方法。AQS获取锁的时候会调用这个，本身就是让子类实现的。
     *
     * 这里判断逻辑是通过(CAS)unsafe.compareAndSwapInt的原子性，来比较并设置值。
     * 最终是比较当前state == 0 ？那么就设置为 1 并返回true。
     *   true: 将当前线程绑定上。return true表示获取锁成功
     *   false: 获取锁失败。
     */
    protected boolean tryAcquire(int unused) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(Thread.currentThread());
            return true;
        }
        return false;
    }

    /**
     * 释放锁，将state置为0
     */
    protected boolean tryRelease(int unused) {
        setExclusiveOwnerThread(null);
        setState(0);
        return true;
    }

    //lock方法也会调用tryAcquire()方法，但是获取失败会中断线程。
    public void lock()        { acquire(1); }
    //尝试获取锁
    public boolean tryLock()  { return tryAcquire(1); }
    public void unlock()      { release(1); }
    public boolean isLocked() { return isHeldExclusively(); }

    /**
     * 结束线程t。如果线程已经start()，并且线程t不为空，也不是中断状态，那么就中断。
     * 这个方法再shutdownNow()中使用了，不获取锁直接中断，
     */
    void interruptIfStarted() {
        Thread t;
        if (getState() >= 0 && (t = thread) != null && !t.isInterrupted()) {
            try {
                t.interrupt();
            } catch (SecurityException ignore) {
            }
        }
    }
}
```

#### Worker存在的意义：
    - 1、控制中断，保证task能够完整执行。防止在shutdown()等情况下意外中断。
    - 2、控制自旋获取getTask()。
    - 3、控制自身退出时机
    
#### Worker如何实现不可重入锁：state (-1,0,1)
    控制中断，主要意义就是尽量保证worker中正在执行的task能成功执行完。
    不要在执行过程中被其他方法intercept()了。
    - 1、继承AQS(AbstractQueuedSynchronizer)实现不可重入锁
    - 2、在new Worker()时,线程还没有start()时,state设置-1。此时线程如果调用intercept()没有意义。
    在runWoker()=>t.start()时再设置state=0
    - 3、在tryLock()->tryAcquire()，使用CAS先比较再重置的方式设置state 0=>1，就保证了不可重入锁。
    因为只有state=0时才能重新设置，其他-1，1的情况都不能设置成功。

#### Worker如何控制中断
    - 1、初始化AQS state = -1，此时不允许调用interrupt()。只有runWorker()将state设置为0才允许调用中断。
    - 2、shutdown()，setMaximumPoolSize()等安全退出或改变线程池的方法，都会调用interruptIdleWorkers();中断空闲worker的方法。
    这个方法中会遍历所有worker，然后尝试获取锁tryLock()，如果tryLock()获取成功，则说明该worker属于空闲状态，
    因为worker自旋时如果获取到task的话就会lock()。只有在getTask()阻塞状态时才会释放锁。故如果w.tryLock()能成功获取锁，
    则说明worker空闲。这一点就是利用不可重入锁的特点来实现的。
    - 3、shutdownNow()，调用的是interruptWorkers()，这个方法是直接遍历worker，直接调用interrupt()，并不会去获取锁。
    但是判断state是不是>-1，因为state=-1时，线程都没有start()呢，没必要中断。

## <span id="jump4">四、ThreadPoolExecutor.runWorker()</span>
![image](https://note.youdao.com/yws/public/resource/49dd430837728c8137d8c1d59164db4e/xmlnote/WEBRESOURCEf922854be5b207842d0c37d69225bcdb/6321)
```java
 /**
 * Main worker run loop.  Repeatedly gets tasks from queue and
 * executes them, while coping with a number of issues:
 * 重复的从队列中获取任务并执行，同时应对一些问题：
 *
 * 1. We may start out with an initial task, in which case we
 * don't need to get the first one. Otherwise, as long as pool is
 * running, we get tasks from getTask. If it returns null then the
 * worker exits due to changed pool state or configuration
 * parameters.  Other exits result from exception throws in
 * external code, in which case completedAbruptly holds, which
 * usually leads processWorkerExit to replace this thread.
 * 我们可能使用一个初始化任务开始，即firstTask为null
 * 然后只要线程池在运行，我们就从getTask()获取任务
 * 如果getTask()返回null，则worker由于改变了线程池状态或参数配置而退出
 * 其它退出因为外部代码抛异常了，这会使得completedAbruptly为true，这会导致在processWorkerExit()方法中替换当前线程
 *
 * 2. Before running any task, the lock is acquired to prevent
 * other pool interrupts while the task is executing, and
 * clearInterruptsForTaskRun called to ensure that unless pool is
 * stopping, this thread does not have its interrupt set.
 * 在任何任务执行之前，都需要对worker加锁去防止在任务运行时，其它的线程池中断操作
 * clearInterruptsForTaskRun保证除非线程池正在stoping，线程不会被设置中断标示
 *
 * 3. Each task run is preceded by a call to beforeExecute, which
 * might throw an exception, in which case we cause thread to die
 * (breaking loop with completedAbruptly true) without processing
 * the task.
 * 每个任务执行前会调用beforeExecute()，其中可能抛出一个异常，这种情况下会导致线程die（跳出循环，且completedAbruptly==true），没有执行任务
 * 因为beforeExecute()的异常没有cache住，会上抛，跳出循环
 *
 * 4. Assuming beforeExecute completes normally, we run the task,
 * gathering any of its thrown exceptions to send to
 * afterExecute. We separately handle RuntimeException, Error
 * (both of which the specs guarantee that we trap) and arbitrary
 * Throwables.  Because we cannot rethrow Throwables within
 * Runnable.run, we wrap them within Errors on the way out (to the
 * thread's UncaughtExceptionHandler).  Any thrown exception also
 * conservatively causes thread to die.
 * 假定beforeExecute()正常完成，我们执行任务
 * 汇总任何抛出的异常并发送给afterExecute(task, thrown)
 * 因为我们不能在Runnable.run()方法中重新上抛Throwables，我们将Throwables包装到Errors上抛（会到线程的UncaughtExceptionHandler去处理）
 * 任何上抛的异常都会导致线程die
 *
 * 5. After task.run completes, we call afterExecute, which may
 * also throw an exception, which will also cause thread to
 * die. According to JLS Sec 14.20, this exception is the one that
 * will be in effect even if task.run throws.
 * 任务执行结束后，调用afterExecute()，也可能抛异常，也会导致线程die
 * 根据JLS Sec 14.20，这个异常（finally中的异常）会生效
 *
 * The net effect of the exception mechanics is that afterExecute
 * and the thread's UncaughtExceptionHandler have as accurate
 * information as we can provide about any problems encountered by
 * user code.
 *
 *
 * @param w the worker
 */
final void runWorker(Worker w) {
//      runWorker()这个方法执行节点是Worker.run()，而run()方法执行节点是addWorker()里的worker.t.start()。
//      所以执行到这个方法的时候，说明线程已经start()了。
    Thread wt = Thread.currentThread();
    Runnable task = w.firstTask;
    w.firstTask = null;

    // 目的是调用worker的tryRelease()方法，将state 置为 0。
    // 意思是当前线程已经运行了，如果有人需要intercept()中断。就可以调用了。
    w.unlock(); // allow interrupts

    //标识线程是否为正常退出的。不抛意想不到的异常就会将这个值置为false。
    //这个标识会传到processWorkerExit()中，在里边判断如果不是正常退出会启动一个新worker来继续处理出现问题的task。
    boolean completedAbruptly = true;
    try {
        //判断firstTask不为空，或者从workQueue阻塞队列中拿到了任务。就就开始执行task。
        //自旋
        while (task != null || (task = getTask()) != null) {
            //开始任务之前先加锁，而且是不可重入的互斥锁。不可重入的目的在Worker中已经说过了。
            //上锁，不是为了防止并发执行任务，为了在shutdown()时不终止正在运行的worker
            w.lock();
            // If pool is stopping, ensure thread is interrupted;
            // if not, ensure thread is not interrupted.  This
            // requires a recheck in second case to deal with
            // shutdownNow race while clearing interrupt
            /**
             * clearInterruptsForTaskRun操作
             * 确保只有在线程stoping时，才会被设置中断标示，否则清除中断标示
             * 1、如果线程池状态>=stop，且当前线程没有设置中断状态，wt.interrupt()
             * 2、如果一开始判断线程池状态<stop，但Thread.interrupted()为true，即线程已经被中断，又清除了中断标示，再次判断线程池状态是否>=stop
             *   是，再次设置中断标示，wt.interrupt()
             *   否，不做操作，清除中断标示后进行后续步骤
             */
            // RUNNING    = -1
            // SHUTDOWN   =  0
            // STOP       =  1
            // TIDYING    =  2
            // TERMINATED =  3
            if (
                    (runStateAtLeast(ctl.get(), STOP) || (Thread.interrupted() && runStateAtLeast(ctl.get(), STOP)))
                     && !wt.isInterrupted()
            )
                wt.interrupt();
            try {
                //任务执行前（子类实现，可自定义操作）
                beforeExecute(wt, task);
                Throwable thrown = null;
                try {
                    //执行任务
                    task.run();
                } catch (RuntimeException x) {
                    thrown = x; throw x;
                } catch (Error x) {
                    thrown = x; throw x;
                } catch (Throwable x) {
                    thrown = x; throw new Error(x);
                } finally {
                    //任务执行后（子类实现，可自定义操作）
                    afterExecute(task, thrown);
                }
            } finally {
                task = null;
                w.completedTasks++;//完成任务数+1
                //解锁
                w.unlock();
            }
        }
        completedAbruptly = false;
    } finally {
        //处理worker的退出
        processWorkerExit(w, completedAbruptly);
    }
}
```
执行流程：

    1、Worker线程启动后，通过Worker类的run()方法调用runWorker(this)
    2、执行任务之前，首先worker.unlock()，将AQS的state置为0，允许中断当前worker线程
    3、开始执行firstTask，调用task.run()，在执行任务前会上锁wroker.lock()，在执行完任务后会解锁，为了防止在任务运行时被线程池一些中断操作中断
    4、在任务执行前后，可以根据业务场景自定义beforeExecute() 和 afterExecute()方法
    5、无论在beforeExecute()、task.run()、afterExecute()发生异常上抛，都会导致worker线程终止，进入processWorkerExit()处理worker退出的流程
    6、如正常执行完当前task后，会通过getTask()从阻塞队列中获取新任务，当队列中没有任务，且获取任务超时，那么当前worker也会进入退出流程

## <span id="jump5">五、ThreadPoolExecutor.getTask()方法</span>
![image](https://note.youdao.com/yws/public/resource/49dd430837728c8137d8c1d59164db4e/xmlnote/WEBRESOURCEcf1176576bd81a00a0bd69bda58f874c/6347)
```java
/**
* Performs blocking or timed wait for a task, depending on
* current configuration settings, or returns null if this worker
* must exit because of any of:
* 1. There are more than maximumPoolSize workers (due to
*    a call to setMaximumPoolSize).
* 2. The pool is stopped.
* 3. The pool is shutdown and the queue is empty.
* 4. This worker timed out waiting for a task, and timed-out
*    workers are subject to termination (that is,
*    {@code allowCoreThreadTimeOut || workerCount > corePoolSize})
*    both before and after the timed wait, and if the queue is
*    non-empty, this worker is not the last thread in the pool.
*
* 执行阻塞队列的：阻塞获取方法：take() 或者 超时等待方法：poll(timeout)方法，取决于当前的配置。
* 如果这个worker返回null，必须满足如下条件：
* 1、超过最大线程数量（因为调用了setMaximumPoolSize()）
* 2、线程池stop了
* 3、线程池shutdown了，并且任务队列queue为空
* 4、这个worker不是核心线程、或者设置了允许核心线程退出。并且超过了keepAliveTime的等待时间，
* 仍然没有获取到task，那么return null
*
*
*  如果返回null，那么worker就是要退出了，所以把工作计数 - 1
* @return task, or null if the worker must exit, in which case
*         workerCount is decremented
*/
private Runnable getTask() {
boolean timedOut = false; // Did the last poll() time out?

for (;;) {
    // 获取线程池状态
    int c = ctl.get();
    int rs = runStateOf(c);

    //线程池状态已经在SHUTDOWN之后了 && (线程池状态在STOP之后了  ||  工作队列为空了)
    //那么返回null，退出线程。并且调用decrementWorkerCount，（CAS)方式核减调当前线程数。
//            RUNNING    = -1 << COUNT_BITS;
//            SHUTDOWN   =  0 << COUNT_BITS;
//            STOP       =  1 << COUNT_BITS;
//            TIDYING    =  2 << COUNT_BITS;
//            TERMINATED =  3 << COUNT_BITS;
    // Check if queue empty only if necessary.
    if (rs >= SHUTDOWN && (rs >= STOP || workQueue.isEmpty())) {
//                （CAS)方式核减调当前线程数。
        //方法里最终是循环调用 ctl.compareAndSet(expect, expect - 1);，直到成功
        decrementWorkerCount();
        return null;
    }

    // 获取当前线程数
    int wc = workerCountOf(c);

    // worker是否允许退出？
    // 设置了允许核心线程退出 || 当前线程数 > 核心线程数
    // Are workers subject to culling?
    boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;

    //(当前线程数 > maximumPollSize || (允许线程退出 && timedOut超时事件过后仍然没有任务))
    // && (线程数 > 1 || 任务队列queue为空)
    if (
            (wc > maximumPoolSize || (timed && timedOut))
            && (wc > 1 || workQueue.isEmpty())
    ) {
        //调用CAS方式，把工作线程数-1，如果-1成功就直接返回null。如果失败就再循环一圈。
        if (compareAndDecrementWorkerCount(c))
            return null;
        continue;
    }

    try {
        //允许线程退出 ？ 调用poll() : 调用take()
        //poll(keepAliveTime)方法，是等待超过keepAliveTime之后会返回null。
        //take()方法，是一直处于阻塞状态，直到队列中有新数据插入时，拿到数据。
        //workQueue的源码分析过，
        // 1、其实就是take()获取时通过线程锁的Condition属性控制线程睡眠挂起await()
        //      Condition notEmpty = lock.newCondition()
        //      notEmpty.await()睡眠等待。
        // 2、在插入时，notEmpty.signal();进行线程唤醒而已。
        Runnable r = timed ?
                workQueue.poll(keepAliveTime, TimeUnit.NANOSECONDS) :
                workQueue.take();
        if (r != null)
            return r;
        //执行到这，意味着当前线程具备退出条件了。下次循环后再结合其他条件确定是否返回null
        timedOut = true;
    } catch (InterruptedException retry) {
        //如果抛异常，那么退出标识就先置为false，继续自旋
        timedOut = false;
    }
}
}
```

核心就是利用workQueue阻塞队列的特性：

    A、workQueue.poll()：如果在keepAliveTime时间内，阻塞队列还是没有任务，返回null
    B、workQueue.take()：如果阻塞队列为空，当前线程会被挂起等待await()；当队列中有任务加入时，线程被唤醒signal()，take方法返回任务
## <span id="jump6">六、ThreadPoolExecutor.processWorkerExit()方法</span>
```java
/**
 * Performs cleanup and bookkeeping for a dying worker. Called
 * only from worker threads. Unless completedAbruptly is set,
 * assumes that workerCount has already been adjusted to account
 * for exit.  This method removes thread from worker set, and
 * possibly terminates the pool or replaces the worker if either
 * it exited due to user task exception or if fewer than
 * corePoolSize workers are running or queue is non-empty but
 * there are no workers.
 *
 * 为被干掉的worker调用清理方法和记录。这个方法只会被worker线程调用。
 * @param w the worker
 * @param completedAbruptly if the worker died due to user exception
 */
private void processWorkerExit(ThreadPoolExecutor.Worker w, boolean completedAbruptly) {
    /**
     * 1、worker数量-1
     * 如果是突然终止，说明是task执行时异常情况导致，即run()方法执行时发生了异常，那么正在工作的worker线程数量需要-1
     * 如果不是突然终止，说明是worker线程没有task可执行了，不用-1，因为已经在getTask()方法中-1了
     */
    //在runWorker中抛异常了才为true，那么就调用CAS方式将工作线程-1
    if (completedAbruptly) // If abrupt, then workerCount wasn't adjusted
        decrementWorkerCount();

    /**
     * 2、从Workers Set中移除worker
     */
    //线程加锁
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock();
    try {
        //已完成任务总数 += worker的完成数
        completedTaskCount += w.completedTasks;
        //将该worker从HashSet中移除
        workers.remove(w);
    } finally {
        mainLock.unlock();
    }

    /**
     * 3、在对线程池有负效益的操作时，都需要“尝试终止”线程池
     * 主要是判断线程池是否满足终止的状态
     * 如果状态满足，但还有线程池还有线程，尝试对其发出中断响应，使其能进入退出流程
     * 没有线程了，更新状态为tidying->terminated
     */
    tryTerminate();

    /**
     * 4、是否需要增加worker线程
     *    1.线程池状态是running 或 shutdown
     *    2.如果当前线程是突然终止的，addWorker()
     *    3.如果当前线程不是突然终止的，但当前线程数量 < 要维护的线程数量，addWorker()
     * 故如果调用线程池shutdown()，直到workQueue为空前，
     * 线程池都会维持corePoolSize个或者1个线程，然后再逐渐销毁这corePoolSize个或者1个线程
     */
    int c = ctl.get();
    //如果状态是running、shutdown，即tryTerminate()没有成功终止线程池，尝试再添加一个worker
    if (runStateLessThan(c, STOP)) {
        //不是突然完成的，即没有task任务可以获取而完成的，计算min，并根据当前worker数量判断是否需要addWorker()
        if (!completedAbruptly) {
            //最小值min = 如果允许核心线程退出，那么最小值就是0。否则就是核心线程数。
            int min = allowCoreThreadTimeOut ? 0 : corePoolSize;
            //如果最小值为0 && 工作队列不为空，那么意味着任务没有消化完，至少还需要一个worker去消化。
            if (min == 0 && ! workQueue.isEmpty())
                min = 1;
            //如果当前的工作worker数量 >= 最小值，就不需要加worker了。否则就会执行下边的addWorker(null,false)方法，添加worker
            if (workerCountOf(c) >= min)
                return; // replacement not needed
        }
        //添加一个没有firstTask的worker
        //只要worker是completedAbruptly突然终止的，或者线程数量小于要维护的数量，就新添一个worker线程。
        //因为抛出异常的情况可能是workQueue的task并没有完成执行完。启动一个空task的worker去消化workQueue队列
        addWorker(null, false);
    }
}
```
## <span id="jump7">七、ThreadPoolExecutor.tryTerminate()方法</span>
```java
/**
 * Transitions to TERMINATED state if either (SHUTDOWN and pool
 * and queue empty) or (STOP and pool empty).  If otherwise
 * eligible to terminate but workerCount is nonzero, interrupts an
 * idle worker to ensure that shutdown signals propagate. This
 * method must be called following any action that might make
 * termination possible -- reducing worker count or removing tasks
 * from the queue during shutdown. The method is non-private to
 * allow access from ScheduledThreadPoolExecutor.
 * 
 * 在以下情况将线程池变为TERMINATED终止状态
 * shutdown 且 正在运行的worker 和 workQueue队列 都empty
 * stop 且  没有正在运行的worker
 * 
 * 这个方法必须在任何可能导致线程池终止的情况下被调用，如：
 * 减少worker数量
 * shutdown时从queue中移除任务
 * 
 * 这个方法不是私有的，所以允许子类ScheduledThreadPoolExecutor调用
 */
final void tryTerminate() {
    //这个for循环主要是和进入关闭线程池操作的CAS判断结合使用的
    for (;;) {
        int c = ctl.get();
         
        /**
         * 线程池是否需要终止
         * 如果以下3中情况任一为true，return，不进行终止
         * 1、还在运行状态
         * 2、状态是TIDYING、或 TERMINATED，已经终止过了
         * 3、SHUTDOWN 且 workQueue不为空
         */
        if (isRunning(c) ||
            runStateAtLeast(c, TIDYING) ||
            (runStateOf(c) == SHUTDOWN && ! workQueue.isEmpty()))
            return;
         
        /**
         * 只有shutdown状态 且 workQueue为空，或者 stop状态能执行到这一步
         * 如果此时线程池还有线程（正在运行任务，正在等待任务）
         * 中断唤醒一个正在等任务的空闲worker
         * 唤醒后再次判断线程池状态，会return null，进入processWorkerExit()流程
         */
        if (workerCountOf(c) != 0) { // Eligible to terminate 资格终止
            interruptIdleWorkers(ONLY_ONE); //中断workers集合中的空闲任务，参数为true，只中断一个
            return;
        }
 
        /**
         * 如果状态是SHUTDOWN，workQueue也为空了，正在运行的worker也没有了，开始terminated
         */
        final ReentrantLock mainLock = this.mainLock;
        mainLock.lock();
        try {
            //CAS：将线程池的ctl变成TIDYING（所有的任务被终止，workCount为0，为此状态时将会调用terminated()方法），期间ctl有变化就会失败，会再次for循环
            if (ctl.compareAndSet(c, ctlOf(TIDYING, 0))) {
                try {
                    terminated(); //需子类实现
                } 
                finally {
                    ctl.set(ctlOf(TERMINATED, 0)); //将线程池的ctl变成TERMINATED
                    termination.signalAll(); //唤醒调用了 等待线程池终止的线程 awaitTermination() 
                }
                return;
            }
        }
        finally {
            mainLock.unlock();
        }
        // else retry on failed CAS
        // 如果上面的CAS判断false，再次循环
    }
}
```
## <span id="jump8">八、ThreadPoolExecutor.interruptIdleWorkers()方法</span>
```java
/**
 * Interrupts threads that might be waiting for tasks (as
 * indicated by not being locked) so they can check for
 * termination or configuration changes. Ignores
 * SecurityExceptions (in which case some threads may remain
 * uninterrupted).
 * 中断在等待任务的线程(没有上锁的)，中断唤醒后，可以判断线程池状态是否变化来决定是否继续
 *
 * @param onlyOne If true, interrupt at most one worker. This is
 * called only from tryTerminate when termination is otherwise
 * enabled but there are still other workers.  In this case, at
 * most one waiting worker is interrupted to propagate shutdown
 * signals in case(以免) all threads are currently waiting.
 * Interrupting any arbitrary thread ensures that newly arriving
 * workers since shutdown began will also eventually exit.
 * To guarantee eventual termination, it suffices to always
 * interrupt only one idle worker, but shutdown() interrupts all
 * idle workers so that redundant workers exit promptly, not
 * waiting for a straggler task to finish.
 * 
 * onlyOne如果为true，最多interrupt一个worker
 * 只有当终止流程已经开始，但线程池还有worker线程时,tryTerminate()方法会做调用onlyOne为true的调用
 * （终止流程已经开始指的是：shutdown状态 且 workQueue为空，或者 stop状态）
 * 在这种情况下，最多有一个worker被中断，为了传播shutdown信号，以免所有的线程都在等待
 * 为保证线程池最终能终止，这个操作总是中断一个空闲worker
 * 而shutdown()中断所有空闲worker，来保证空闲线程及时退出
 */
private void interruptIdleWorkers(boolean onlyOne) {
    final ReentrantLock mainLock = this.mainLock;
    mainLock.lock(); //上锁
    try {
        for (Worker w : workers) { 
            Thread t = w.thread;
            //w.tryLock()，只有执行完task，正在getTask()阻塞状态的worker才能获取到lock。因为runWorker()时获取到task,会先lock()
            if (!t.isInterrupted() && w.tryLock()) {
                try {
                    t.interrupt();
                } catch (SecurityException ignore) {
                } finally {
                    w.unlock();
                }
            }
            if (onlyOne)
                break;
        }
    } finally {
        mainLock.unlock(); //解锁
    }
}
```
