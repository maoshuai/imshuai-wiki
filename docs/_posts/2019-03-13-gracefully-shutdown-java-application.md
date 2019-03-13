---
title: 优雅关闭Java 应用
tags: Java 线程池
---

Java 开发的后台应用，通常是保持7x24小时运行的，但遇到版本变更或其他原因，需要停止应用，如果粗暴的停止，可能导致交易损失。如何避免停机带来的交易损失呢？下面是我最近的项目的实践经验总结。

总的思想是：**从最外层依次向内层，逐层关闭服务**。
<!--more-->

# 离开集群

应用一般是集群部署的，前面会有负载均衡或路由负责交易的分发。因此，我们首先要将应用从集群中摘除。以我司使用的F5负载均衡设备为例，F5会发送心跳报文到每台应用的服务端口，用来判断服务状态，当超过一定次数（比如3次）返回失败，则F5将其剔除集群，不再发送交易，直到下一次探测报文返回成功。

除了应用本身的异常外，**我们希望应用有办法做到主动离开集群** 。比如让心跳报文的返回，根据某一个可控的状态变化。最简单的是，在某个目录设置一个状态文件，心跳时检测此文件，当该文件存在时则返回失败报文。这样，我们可以通过创建状态文件，让应用撤离集群。

这一步的目的，是将交易入口关闭，但系统实际功能仍完好，即虽然撤离了集群，但对于进来和正在处理的交易都能正常处理。

# 关闭渠道

后台应用一般通过网络渠道接入，可以是HTTP、TCP或者MQ等。接下来可以将渠道关闭，比如TCP关闭socket，MQ关闭通道。进一步保证不会有交易进入。

这一步还可以配合检测步骤，比如MQ队列已经为空，保证不会有交易进入。

# 关闭线程池

后台应用一般维护线程池处理任务。虽然没有交易进入，但之前进入的交易，可能仍然在线程里运行。这个时候直接杀掉进程，可能导致线程池中未结束的交易异常。

但对于 Java 的线程，是没有绝对的办法从外部停止的。因此，我们通常采用等待的办法。如果使用的是`ExecutorService`，则可以调用`shutdown`方法关闭线程池，然后调用`awaitTermination`等待充分长的时间，正常情况下都可以就结束。特殊情况下没有结束，也只能调用`shutdownNow`方法强制结束（即便如此，也只是best-effort的操作）。

# 关闭Spring容器和其他依赖

线程池关闭后，就不会有任何业务逻辑在运行了，这时候可以放心的关闭Spring容器或其他依赖。如果线程池本身也是在由Spring容器管理的，则可以通过Spring的`SmartLifeCycle`实现关闭顺序：即保证先关闭和销毁线程池的bean，再销毁其他bean，最后关闭Spring容器。

具体来说，让涉及线程池的bean实现`SmartLifecycle`接口，并让`getPhase()`方法返回一个足够大的值（即第一个被关闭），然后在`stop()`方法内实现线程池的关闭逻辑。

# 补充说明

## shutdown和shutdownNow的比较
`shutdown`和`shutdownNow`都是停止线程池，但仍有不同，相比而言，shutdown方法更为优雅，若不过分追求停止的速度，应首选shutdown。

区别：

* shutdown

首先拒绝新提交的任务，但对于已提交到线程池的任务（包括执行中的，以及在队列中的）仍然耐心等待处理结束。因此不用担心已提交的任务被终止。

但如果线程池本身在队列里积压了很多任务，可能要等很久才能停止。

* shutdownNow

除了拒绝新提交的任务外，还会终止正在执行的任务，而处于队列中的任务更不用说了，也会终止。

相同：

* 都不能绝对保证线程池的任务被终止，这是由线程的机制决定的。
* 都是异步方法，即这两个方法都是立马返回，并不阻塞。如果要等待结束，需调用`awaitTermination`方法。一个常见的错误，就是当成同步方法调用，导致还停止线程池就做了下一步操作。

一个最佳实践是，先调用shutdown，如果awaitTermination超时还没有结束，再调用shutdownNow。当然shutdownNow还有可能超时未结束，这种情况只能强制停止了，比如外部kill -9结束。下面是一个停止线程池的示例代码：
```java
executor.shutdown();
if (!executor.awaitTermination(SHUTDOWN_TIME)) { 
    Logger.log("Executor did not terminate in the specified time."); 
    List<Runnable> droppedTasks = executor.shutdownNow(); 
    Logger.log("Executor was abruptly shut down. " + droppedTasks.size() + " tasks will not be executed."); 
}
```
## 使用Shtudown hook
关闭应用时，我们可以通过`kill` 命令向java进程发送默认的`TERM`信号。为了能捕捉这个信号，并完成上述步骤，我们需要向JVM注册Shtudown hook。

```java
Runtime.getRuntime().addShutdownHook(new Thread() {
    public void run() {
        // do stop things
    }
});
```

如果是Spring管理的，可以对Spring容器注册shutdown hook：
```java
AbstractApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
context.registerShutdownHook();
```

## kill 命令

正常我们使用kill命令，发送默认的`TERM`信号，可以触发JVM的shutdown hook。如果遇到特殊情况，一直结束不了，可能只能使用暴力的`kill -9`了。

# See also

* [Graceful shutdown of threads and executor](https://stackoverflow.com/questions/3332832/graceful-shutdown-of-threads-and-executor)
* [How to setup a fixed shutdown sequence with Spring](http://www.gitshah.com/2014/04/how-to-setup-fixed-shutdown-sequence.html)