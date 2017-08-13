---
layout: post
title:  "JAVA Multithreading"
date:   2017-08-13 16:54:13
tags: JAVA
author: Temi Lee
---

**在JDK1.5 中引入了一些新的并发API，位于java.util.concurrent包下，本篇博客旨在介绍本包下一些常用的API**
<br/><br/>

**Java 自带的线程池:Executor**
为什么要使用线程池:
- 线程复用，节省创建和销毁线程的消耗
- 方便对需要执行的task进行统一的管理
- 手动创建Thread类有安全风险(比如在构造方法中进行this调用，将会暴漏一个未初始化完成的Thread类对象)

Executor 框架包括:Executor，Executors，ExecutorService，CompletionService，Future，Callable

- Executor: 线程池的定义接口,`源码和翻译:`

{% highlight java %}

    package java.util.concurrent;

    /**
     * An object that executes submitted {@link Runnable} tasks. This
     * interface provides a way of decoupling task submission from the
     * mechanics of how each task will be run, including details of thread
     * use, scheduling, etc.  An <tt>Executor</tt> is normally used
     * instead of explicitly creating threads. For example, rather than
     * invoking <tt>new Thread(new(RunnableTask())).start()</tt> for each
     * of a set of tasks, you might use:
     *
     *这是一个用于执行提交的任务的接口定义，这个接口解除了任务的提交和系统的具体执行策略之间的耦合
     *程序中应当使用Executor 替代显示的创建线程
     *比如你应当如下执行任务，而不是调用new Thread(new(RunnableTask())).start()
     *
     * <pre>
     * Executor executor = <em>anExecutor</em>;
     * executor.execute(new RunnableTask1());
     * executor.execute(new RunnableTask2());
     * ...
     * </pre>
     *
     * However, the <tt>Executor</tt> interface does not strictly
     * require that execution be asynchronous. In the simplest case, an
     * executor can run the submitted task immediately in the caller's
     * thread:
     *
     *然而，并不是所有的Executor实现都是严格的异步执行任务的，比如，一个Executor
     *可以在调用方的线程中同步的执行任务，像下面这样：
     *
     * <pre>
     * class DirectExecutor implements Executor {
     *     public void execute(Runnable r) {
     *         r.run();
     *     }
     * }</pre>
     *
     * More typically, tasks are executed in some thread other
     * than the caller's thread.  The executor below spawns a new thread
     * for each task.
     *
     *更典型的，提交的任务可以在任务提交者之外的一些线程中去执行
     *下面的这个executor实现，将会为每个提交的任务都开启一个新的线程去执行
     *
     * <pre>
     * class ThreadPerTaskExecutor implements Executor {
     *     public void execute(Runnable r) {
     *         new Thread(r).start();
     *     }
     * }</pre>
     *
     * Many <tt>Executor</tt> implementations impose some sort of
     * limitation on how and when tasks are scheduled.  The executor below
     * serializes the submission of tasks to a second executor,
     * illustrating a composite executor.
     *
     *很多Executor的实现 增加了一些对于任务调用方式和时机的限制，下面的这个Executor顺序执
     *提交任务的另外一个Executor去执行
     *
     *  <pre> {@code
     * class SerialExecutor implements Executor {
     *   final Queue<Runnable> tasks = new ArrayDeque<Runnable>();
     *   final Executor executor;
     *   Runnable active;
     *
     *   SerialExecutor(Executor executor) {
     *     this.executor = executor;
     *   }
     *
     *   public synchronized void execute(final Runnable r) {
     *     tasks.offer(new Runnable() {
     *       public void run() {
     *         try {
     *           r.run();
     *         } finally {
     *           scheduleNext();
     *         }
     *       }
     *     });
     *     if (active == null) {
     *       scheduleNext();
     *     }
     *   }
     *
     *   protected synchronized void scheduleNext() {
     *     if ((active = tasks.poll()) != null) {
     *       executor.execute(active);
     *     }
     *   }
     * }}</pre>
     *
     * The <tt>Executor</tt> implementations provided in this package
     * implement {@link ExecutorService}, which is a more extensive
     * interface.  The {@link ThreadPoolExecutor} class provides an
     * extensible thread pool implementation. The {@link Executors} class
     * provides convenient factory methods for these Executors.
     *
     *ExecutorService 扩充了一些Executor的表现。ThreadPoolExecutor 提供了一个
     *可扩展的线程池的实现，Executors 为一给线程池实现提供了方便的工程方法
     *
     * <p>Memory consistency effects: Actions in a thread prior to
     * submitting a {@code Runnable} object to an {@code Executor}
     * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
     * its execution begins, perhaps in another thread.
     *
     * 内存一致性影响：
     * TODO
     * @since 1.5
     * @author Doug Lea
     */
    public interface Executor {

        /**
         * Executes the given command at some time in the future.  The command
         * may execute in a new thread, in a pooled thread, or in the calling
         * thread, at the discretion of the <tt>Executor</tt> implementation.
         *
         *在合适的时机执行提交的task，是在新的线程、池线程、调用者线程中那个执行取决于
         *具体的线程池实现
         *
         * @param command the runnable task
         * @throws RejectedExecutionException if this task cannot be
         * accepted for execution.
         * @throws NullPointerException if command is null
         */
        void execute(Runnable command);
    }

{% endhighlight %}

- Executors 为Executor提供一些帮助和工厂方法，内部最终调用ThreadPoolExecutor线程池实现类
另外还提供了 Runnable 接口转Callable的一些工具方法
三个创建线程池的方法:
- newFixedThreadPool  : 创建固定数目线程的线程池。
- newCachedThreadPool : 创建一个可缓存的线程池。
- newSingleThreadExecutor : 创建一个单线程化的Executor。

- ThreadPoolExecutor 线程池的实现类，类结构如下:
![ThreadPoolExecutor类图][1]

构造方法:

{% highlight java %}
    /**
     * Creates a new {@code ThreadPoolExecutor} with the given initial
     * parameters.
     *
     * @param corePoolSize the number of threads to keep in the pool, even
     *        if they are idle, unless {@code allowCoreThreadTimeOut} is set
     *
     * corePoolSize:线程池中保存的线程数量的基数
     *
     * @param maximumPoolSize the maximum number of threads to allow in the
     *        pool
     *
     * maximumPoolSize 线程池中所允许的最大线程数量
     *
     * @param keepAliveTime when the number of threads is greater than
     *        the core, this is the maximum time that excess idle threads
     *        will wait for new tasks before terminating.
     *
     *keepAliveTime 当线程池的线程数量大于线程池的线程基数(corePoolSize)的时候
     *这个时间长度指示空闲线程在被总之之前等待任务的最大时间
     *
     * @param unit the time unit for the {@code keepAliveTime} argument
     *
     * keepAliveTime参数的时间单位
     *
     * @param workQueue the queue to use for holding tasks before they are
     *        executed.  This queue will hold only the {@code Runnable}
     *        tasks submitted by the {@code execute} method.
     *
     * workQueue 任务队列，在任务执行之前，保存着由execute方法提交的实现了Runnable接口的任务
     *
     * TIPS: 提交Runnbale任务使用submit方法
     *
     * @param threadFactory the factory to use when the executor
     *        creates a new thread
     *
     * threadFactory 创建线程的工厂
     *
     * @param handler the handler to use when execution is blocked
     *        because the thread bounds and queue capacities are reached
     *
     *handler 当线程池的线程都是繁忙状态并且workQueue 放满，执行此策略拒绝新的任务
     *
     * @throws IllegalArgumentException if one of the following holds:<br>
     *         {@code corePoolSize < 0}<br>
     *         {@code keepAliveTime < 0}<br>
     *         {@code maximumPoolSize <= 0}<br>
     *         {@code maximumPoolSize < corePoolSize}
     * @throws NullPointerException if {@code workQueue}
     *         or {@code threadFactory} or {@code handler} is null
     */
    public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
{% endhighlight %}


[1]: /img/blog/multithreading/multithreading.png

