# EventLoop初始化
上一节讲到NioEnventLoopGroup初始化时，实际是初始化MultithreadEventExecutorGroup内部维护的一个EventExecutor[]数组，并且该数组中实际的EventExecutor实现是NioEventLoop，那么我们就来看看NioEventLoop实例化都做了什么操作。

----
## NioEventLoop类图
![NioEventLoop类图]()

NioEventLoop继承于SingleThreadEventLoop，从字面上来看就知道，这是一个单线程，也就是我们平时所说的Nio线程。

## 初始化流程
初始化时调用了NioEventLoop的构造函数
``` java
NioEventLoop(NioEventLoopGroup parent, Executor executor, SelectorProvider selectorProvider,SelectStrategy strategy) {
        super(parent, executor, false);
        if (selectorProvider == null) {
            throw new NullPointerException("selectorProvider");
        }
        if (strategy == null) {
            throw new NullPointerException("selectStrategy");
        }
        provider = selectorProvider;
        selector = openSelector();
        selectStrategy = strategy;
    }
```
从上面可以看到，NioEventLoop的selector局部变量得到初始化，调用的是provider的openSelector()方法，然后继续看调用父类的构造函数，在SingleThreadEventExecutor中做了实际的初始化操作。
``` java
protected SingleThreadEventExecutor(EventExecutorGroup parent, Executor executor, boolean addTaskWakesUp) {
        super(parent);

        if (executor == null) {
            throw new NullPointerException("executor");
        }

        this.addTaskWakesUp = addTaskWakesUp;
        this.executor = executor;
        taskQueue = newTaskQueue();
    }
```
这里会判断executor是否为空，结合前面的NioEventLoopGroup部分代码，这个executor实际上是ThreadPerTaskExecutor的实例，它的execute方法是用于新建一个线程并启动。
``` java
public final class ThreadPerTaskExecutor implements Executor {
    private final ThreadFactory threadFactory;

    public ThreadPerTaskExecutor(ThreadFactory threadFactory) {
        if (threadFactory == null) {
            throw new NullPointerException("threadFactory");
        }
        this.threadFactory = threadFactory;
    }

    @Override
    public void execute(Runnable command) {
        threadFactory.newThread(command).start();
    }
}
```
接着初始化了一个任务队列taskQueue（LinkedBlockingQueue），调用NioEventLoop的execute方法就会往这个队列添加任务。

到这里NioEventLoop就完成了初始化。