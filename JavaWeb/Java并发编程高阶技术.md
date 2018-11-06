## 第1章 课程介绍
#### 1-3 并发编程框架Disruptor与BlockingQueue压力测试性能对比
```
public class ArrayBlockingQueue4Test {

    public static void main(String[] args) {
        final ArrayBlockingQueue<Data> queue = new ArrayBlockingQueue<Data>(100000000);
        final long startTime = System.currentTimeMillis();
        new Thread(() -> {
            long i = 0;
            while (i < Constants.EVENT_NUM_OHM) {
                Data data = new Data(i, "c" + i);
                try {
                    queue.put(data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                i++;
            }
            System.out.println("i = " + i);
        }).start();

        new Thread(() -> {
            int k = 0;
            while (k < Constants.EVENT_NUM_OHM) {
                try {
                    queue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                k++;
            }
            System.out.println("k = " + k);
            long endTime = System.currentTimeMillis();
            System.out.println("ArrayBlockingQueue costTime = " + (endTime - startTime) + "ms");
        }).start();
    }

}
```
```
public class DisruptorSingle4Test {

    public static void main(String[] args) {
        int ringBufferSize = 65536;
        final Disruptor<Data> disruptor = new Disruptor<Data>(
                () -> new Data(),
                ringBufferSize,
                Executors.newSingleThreadExecutor(),
                ProducerType.SINGLE,
                new YieldingWaitStrategy()
        		);

        DataConsumer consumer = new DataConsumer();
        disruptor.handleEventsWith(consumer);
        disruptor.start();
        new Thread(() -> {
            RingBuffer<Data> ringBuffer = disruptor.getRingBuffer();
            for (long i = 0; i < Constants.EVENT_NUM_OHM; i++) {
                long seq = ringBuffer.next();
                Data data = ringBuffer.get(seq);
                data.setId(i);
                data.setName("c" + i);
                ringBuffer.publish(seq);
            }
        }).start();
    }

}
```
```
ArrayBlockingQueue costTime = 15393ms
Disruptor costTime = 7914ms
```
结论：Disruptor性能远远高于BlockingQueue

## 第2章 并发编程框架核心讲解
#### 2-2 并发编程框架-QuickStart-基础元素工厂类
**编程模型**
- 建立一个工厂Event类，用于创建Event类实例对象
- 需要有一个监听事件类，用于处理数据（Event类）
- 实例化Disruptor实例，配置一系列参数，编写Disruptor核心组件
- 编写生产者组件，向Disruptor容器中去投递数据

创建disruptor工程、引入依赖，代码具体见quickstart包
OrderEvent、OrderEventFactory实现第一步
```
public class OrderEvent {

	private long value; //订单的价格

	public long getValue() {
		return value;
	}

	public void setValue(long value) {
		this.value = value;
	}
	
}
```
```
public class OrderEventFactory implements EventFactory<OrderEvent>{

	public OrderEvent newInstance() {
		return new OrderEvent();		//这个方法就是为了返回空的数据对象（Event）
	}

}
```
#### 2-3 并发编程框架-QuickStart-消费端事件处理器
OrderEventHandler
```
public class OrderEventHandler implements EventHandler<OrderEvent>{

	public void onEvent(OrderEvent event, long sequence, boolean endOfBatch) throws Exception {
		Thread.sleep(Integer.MAX_VALUE);
		System.err.println("消费者: " + event.getValue());
	}

}
```

#### 2-4 并发编程框架-QuickStart-构建Disruptor实例
Main
```
    public static void main(String[] args) {
        // 参数准备工作
        OrderEventFactory orderEventFactory = new OrderEventFactory();
        int ringBufferSize = 4;
        ExecutorService executor = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

        /**
         * 1 eventFactory: 消息(event)工厂对象
         * 2 ringBufferSize: 容器的长度
         * 3 executor: 线程池(建议使用自定义线程池) RejectedExecutionHandler
         * 4 ProducerType: 单生产者 还是 多生产者
         * 5 waitStrategy: 等待策略
         */
        //1. 实例化disruptor对象
        Disruptor<OrderEvent> disruptor = new Disruptor<OrderEvent>(orderEventFactory,
                ringBufferSize,
                executor,
                ProducerType.SINGLE,
                new BlockingWaitStrategy());

        //2. 添加消费者的监听 (构建disruptor 与 消费者的一个关联关系)
        disruptor.handleEventsWith(new OrderEventHandler());

        //3. 启动disruptor
        disruptor.start();

        //4. 获取实际存储数据的容器: RingBuffer
        RingBuffer<OrderEvent> ringBuffer = disruptor.getRingBuffer();
        OrderEventProducer producer = new OrderEventProducer(ringBuffer);
        ByteBuffer bb = ByteBuffer.allocate(8);
        for (long i = 0; i < 5; i++) {
            bb.putLong(0, i);
            producer.sendData(bb);
        }

        disruptor.shutdown();
        executor.shutdown();
    }
```

#### 2-5 并发编程框架-QuickStart-生产者组件投递数据
```
public class OrderEventProducer {

	private RingBuffer<OrderEvent> ringBuffer;
	
	public OrderEventProducer(RingBuffer<OrderEvent> ringBuffer) {
		this.ringBuffer = ringBuffer;
	}
	
	public void sendData(ByteBuffer data) {
		//1 在生产者发送消息的时候, 首先 需要从我们的ringBuffer里面 获取一个可用的序号
		long sequence = ringBuffer.next();	//0	
		try {
			//2 根据这个序号, 找到具体的 "OrderEvent" 元素 注意:此时获取的OrderEvent对象是一个没有被赋值的"空对象"
			OrderEvent event = ringBuffer.get(sequence);
			//3 进行实际的赋值处理
			event.setValue(data.getLong(0));			
		} finally {
			//4 提交发布操作
			ringBuffer.publish(sequence);			
		}
	}

}
```

#### 2-6 并发编程框架Disruptor-核心机制-生产消费模型
- 初看Disruptor，给人的印象就是RingBuffer是其核心，生产者向RingBuffer中写入元素，消费者从RingBuffer中消费元素
- RingBuffer是啥?
它是一个环，首尾相接的环
它用做在不同上下文（线程）间传递数据的buffer！
RingBuffer拥有一个序号，这个序号指向数组中下一个可用元素

#### 2-7 并发编程框架Disruptor-仍芝麻与捡芝麻小故事
- 生产者和消费者的故事
- 有一个数组：生产者往里面扔芝麻，消费者从里面捡芝麻
- 但是扔芝麻和捡芝麻也要考虑速度的问题
1. 消费者捡的比扔的快，那么消费者要停下来，生产者扔了新的芝麻，然后消费者继续
2. 数组的长度是有限的，生产者到末尾的时候会再从数组的开始位置继续。这个时候可能会追上消费者，消费者还没从那个地方捡走芝麻，这个时候生产者要等待消费者捡走芝麻，然后继续
3. 随着你不停的填充这个buffer，这个序号会一直增长，直到绕过这个环
4. 要找到数组中当前序号指向的元素，可以通过mod操作：sequence mod array length = array index
5. 槽的个数是2的N次方更有利于基于二进制的计算机进行计算

#### 2-8 并发编程框架Disruptor-核心-RingBuffer
- RingBuffer：基于数组的缓存实现，也是创建sequencer与定义WaitStrategy的入口
- Disruptor：持有RingBuffer、消费者线程池Executor、消费者集合ConsumerRepository等引用

#### 2-9 并发编程框架Disruptor-核心-Sequence、Sequencer、SequenceBarrier
**Sequence**
- 通过顺序递增的序号来编号，管理进行交换的数据
- 对数据的处理过程总是沿着序号逐个递增处理
- 一个Sequence用于跟踪标识某个特定的事件处理者（RingBuffer/Producer/Consumer）的处理进度
- Sequence可以看成是一个AtomicLong用于标识进度
- 还有另外一个目的就是防止不同Sequence之间CPU缓存伪共享的问题

**Sequencer**
- Sequencer是Disruptor的真正核心
- 此接口有连个实现类：SingleProducerSequencer、MultiProducerSequencer
- 主要实现生产者和消费者之间快速、正确地传递数据的并发算法

**Sequence Barrier**
- 用于保持对RingBuffer的Main Published Sequence和Consumer之间的平衡关系，Sequence Barrier还定义了决定Consumer是否还有可处理的事件的逻辑

#### 2-10 并发编程框架Disruptor-核心-WaitStrategy消费者等待策略
**WaitStrategy**
- 决定一个消费者将如何等待生产者将Event置入Disruptor！
- 主要策略有：BlockingWaitStrategy、SleepingWaitStrategy、YieldingWaitStrategy
- BlockingWaitStrategy是最低效的策略，但其对CPU的消耗最小并且在各种不同部署环境中能提供更加一致的性能表现
- SleepingWaitStrategy的性能表现跟BlockingWaitStrategy差不多，对CPU的消耗也类似，但其对生产者线程的影响最小，适合用于异步日志类似的场景
- YieldingWaitStrategy的性能是最好的，适合用于低延迟的系统。在要求极高性能且事件处理线数小于CPU逻辑核心数的场景中，推荐使用此策略；例如，CPU开启超线程的特性

#### 2-11 并发编程框架Disruptor-核心-EventProcessor，WorkProcessor等
**Event**
- 从生产者到消费者过程中所处理的数据单元
- Disruptor中没有代码表示Event，因为它完全是由用户定义的

**EventProcessor**
- 主要事件循环，处理Disruptor中的Event，拥有消费者的Sequence
- 它有一个实现类是BatchEventProcessor，包含了event loop有效的实现，并且将回调到一个EventHandler接口的实现对象

**EventHandler**
由用户实现并且代表了Disruptor中的一个消费者的接口，也就是我们的消费者逻辑都需要写在这里

**WorkProcessor**
确保每个sequence只被一个processor消费，在同一个WorkPool中处理多个WorkProcessor不会消费同样的sequence

#### 2-12 并发编程框架Disruptor-核心概念整体图解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-cd43e315727c38be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第3章 并发编程框架高级特性讲解 
#### 3-2 互联网大厂核心链路方案详解
- 啥是核心链路？
- 核心链路的代码实现，往往业务逻辑非常复杂
- 核心链路特点：至关重要且业务复杂，那么代码应该如何实现呢？
实现方式一：传统的完全解耦模式
实现方式二：模板模式
- 解决手段：
1. 领域模型的高度抽象！
2. 寻找更好的框架帮助我们进行编码！
- 使用框架：
1. 有限状态机框架，例如Spring-StateMachine！
2. 使用Disruptor

#### 3-4 串、并行操作实战应用
- handleEventsWith 方法
- 串行操作：使用链式调用的方式
- 并行操作：使用单独调用的方式

heigh包下的Main
```
		disruptor
		.handleEventsWith(new Handler1())
		.handleEventsWith(new Handler2())
		.handleEventsWith(new Handler3());
```
```
		disruptor.handleEventsWith(new Handler1(), new Handler2(), new Handler3());
//		disruptor.handleEventsWith(new Handler2());
//		disruptor.handleEventsWith(new Handler3());
```
CountDownLatch感觉很好用！！！

#### 3-6 菱形操作
- Disruptor可以实现串并行同时编码
```
		disruptor.handleEventsWith(new Handler1(), new Handler2())
		.handleEventsWith(new Handler3());

		EventHandlerGroup<Trade> ehGroup = disruptor.handleEventsWith(new Handler1(), new Handler2());
		ehGroup.then(new Handler3());
```

#### 3-7 多边形操作与底层代码深度解析
- 单消费者多边形操作，线程数需要大于EventHandle数
```
		Handler1 h1 = new Handler1();
		Handler2 h2 = new Handler2();
		Handler3 h3 = new Handler3();
		Handler4 h4 = new Handler4();
		Handler5 h5 = new Handler5();
		disruptor.handleEventsWith(h1, h4);
		disruptor.after(h1).handleEventsWith(h2);
		disruptor.after(h4).handleEventsWith(h5);
		disruptor.after(h2, h5).handleEventsWith(h3);
```

#### 3-8 多生产者多消费者实战应用-1
- 依赖Disruptor配置实现多生产者：create方法
- 依赖WorkerPool实现多消费者：workerPool构造函数

代码见：heigh.multi包

#### 3-12 本章小结
- 本章主要对互联网大厂中如何去进行核心链路编码做了一个简要的解释，我们可以使用Disruptor进行异步化，串行化结合化的方式去进行编码，从而提高我们应用服务的性能，实现高效的落地！并且对Disruptor多生产者多消费者模型进行了学习！

## 第4章 并发编程深入学习与面试精讲
#### 4-2 并发编程面试-并发类容器核心
- ConcurrentMap
ConcurrentHashMap
ConcurrentSkipListMap
- CopyOnWrite
CopyOnWriteArrayList
- ArrayBlockingQueue、LinkedBlockingQueue
- SynchronousQueue、PriorityBlockingQueue
- DelayQueue

代码见：Review.java

#### 4-3 并发编程面试-Volatile与内存分析
- Volatile
作用一：多线程间的可见性
作用二：阻止指令重排序

#### 4-4 并发编程面试-Atomic系列类与UnSafe
- Atomic系列类提供了原子性操作，保障多线程下的安全
- UnSafe类的四大作用：
内存操作
字段的定位和修改
挂起与恢复
CAS操作（乐观锁）

#### 4-5 并发编程面试-J.U.C常用工具类
- CountDownLatch & CyclicBarrier
- Future模式与Caller接口
- Exchanger线程数据交换器
- ForkJoin并行计算框架
- Semaphore信号量

#### 4-6 并发编程面试-AQS各种锁
- ReentrantLock重入锁
- ReentrantReadWriteLock读写锁
- Condition条件判断
- LockSupport基于线程的锁

#### 4-7 并发编程面试-线程池最佳使用指南
- Executors工厂类（不建议使用）
- ThreadPoolExecutor自定义线程池（建议）
- 计算密集型与IO密集型
- 如何正确的使用线程池：合理关闭

#### 4-8 并发编程面试-AQS架构核心
- AQS：抽象类
- AQS维护了一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）
- AQS定义两种资源共享方式：Exclusive、Share
- isHeldExclusively方法：该线程是否正在独占资源
- tryAcquire / tryRelease：独占的方式尝试获取和释放资源
- tryAcquireShared / tryReleaseShared：共享的方式尝试获取和释放资源

#### 4-9 并发编程面试-ReentrantLock底层原理分析
- 以ReentrantLock重入锁为例，state初始化为0，表示未锁定状态
- A线程lock()时，会调用tryAcquire()独占该锁并将state+1
- 此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁
- 当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念
- 但要注意，获取多少次就要释放多少次，这样才能保证state是能回到0态的

#### 4-10 并发编程面试-ReentrantLock底层源码深度解析
ReentrantLock 源码

#### 4-12 并发编程面试-CountDownLatch底层原理分析
- 以CountDownLatch为例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）
- 这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS减1
- 等到所有子线程都执行完后（即state=0），会unpark()调用线程，
- 然后主调用线程就会从await()函数返回，继续后续动作

#### 4-13 本章小结
- 这章主要针对并发编程的技术内容和大家进行技术点的扫盲，目的就是为了巩固大家的基础功底，在面试的时候能够起到作用！并且通过阅读底层代码，加深大家对并发编程的理解，最主要的就是为了下一章节我们分析底层代码做一个基础铺垫！

## 第5章 并发编程框架底层源码深度分析
#### 5-2 并发编程框架Disruptor-整体架构UML类图分析
Disruptor核心流程UML

#### 5-3 并发编程框架Disruptor-为何它的底层性能如此牛掰
- 数据结构层面：使用环形结构、数组、内存预加载
- 使用单线程写方式、内存屏障
- 消除伪共享（填充缓存行）
- 序号栅栏和序号配合使用来消除锁和CAS

#### 5-4 并发编程框架Disruptor-数据结构设计原理与底层源码深度分析
- RingBuffer使用数组Object[] entries作为存储元素
- 内存预加载机制
RingBuffer源码分析

#### 5-5 并发编程框架Disruptor-单线程写核心架构思想
- Disruptor的RingBuffer，之所以可以做到完全无锁，也是因为单线程写，这是所有前提的前提
- 离了这个前提条件，没有任何技术可以做到完全无锁
- Redis、Netty等等高性能技术框架的设计都是这个核心思想

#### 5-6 并发编程框架Disruptor-系统级别内存屏障实现
- 要正确的实现无锁，还需要另外一个关键技术：内存屏障
- 对应到Java语言，就是valotile变量与happens before语义
- 内存屏障 —— Linux的smp_wmb()/smp_rmb()
- 系统内核：拿Linux的kfifol来举例：smp_wmv()，无论是底层的读写都是使用了Linux的smp_wmb

#### 5-7 并发编程框架Disruptor-填充缓存行消除伪共享机制来提升性能
- 缓存系统中是以缓存行（cache line）为单位存储的
- 缓存行是2的整数幂个连续字节，一般为32-256个字节
- 最常见的缓存行大小是64个字节
- 当多线程修改相互独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享
- 防止不同Sequence之间CPU缓存伪共享的问题

#### 5-8 并发编程框架Disruptor-序号栅栏机制底层代码深度分析
- 我们在生产者进行投递Event的时候，总是会使用：long sequence = ringBuffer.next();
- Disruptor3.0中，序号栅栏SequenceBarrier和Sequence搭配使用协调和管理消费者与生产者的工作节奏，避免了锁和CAS的使用
- 在Disruptor3.0中，各个消费者和生成者持有自己的序号，这些序号的变化必须满足如下基本条件：
- 消费者序号数值必须小于生产者序号数值
- 消费者序号数值必须小于其前置消费者的序号数值
- 生产者序号数值不能大于消费者中最小的序号数值，以避免生产者速度过快，将还未来得及消费的消息覆盖

**ringBuffer.next() 源码分析：高性能阻塞机制**

#### 5-11 WaitStrategy等待策略底层源码深度分析
- Disruptor之所以说是高性能，其实也有一部分原因取决于等待策略的实现：WaitStrategy接口
BlockingWaitStrategy 源码分析

#### 5-12 EventProcessor核心架构设计与底层源码深度分析
- EventProcessor实现类：BatchEventProcessor代码核心分析

#### 5-13 本章小结
- 本章节主要对Disruptor底层核心实现进行详细的分析与探讨，通过了解和学习底层，我们能获得很多启发和收获，希望大家能够在以后的工作和实战中，使用这些优秀的设计思路，最终成为并发编程大牛！

## 第6章 Netty整合并发编程框架Disruptor实战百万长链接服务构建
#### 6-2 Disruptor与Netty整合实现百万长链接接入_环境构建
- 在使用Netty进行接收处理数据的时候，我们尽量都不要在工作线程上去编写自己的代码逻辑
- 我们需要利用异步的机制，不如使用线程池异步处理，如果使用线程池就意味着使用阻塞队列，这里可以替换为Disruptor提高性能
disruptor-netty-client
disruptor-netty-server
disruptor-netty-common

#### 6-3 Disruptor与Netty整合-服务端代码最佳实现
TranslatorData、NettyServer、ServerHandler
TCP握手原理与Netty线程组的结合：sync + accept = backlog
MarshallingCodeCFactory编解码

#### 6-6 Disruptor与Netty整合-客户端代码最佳实现
NettyClient、ClientHandler、NettyClientApplication

#### 6-9 Netty的高性能之道核心问题分析-异步化处理业务
- 在使用Netty进行接收处理数据的时候，我们尽量都不要在工作线程上全编写自己的代码逻辑
- 我们需要利用异步的机制，比如使用线程池异步处理，如果使用线程池就意味着使用阻塞队列，这里可以替换为Disruptor提高性能

#### 6-10 Disruptor核心池化封装实现
- 构建Netty高性能的核心架构图
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d439174707680e2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
MessageConsumer、MessageProducer、RingBufferWorkerPoolFactory、

#### 6-13 高性能服务端与客户端落地实现承载百万级连接接入
ServerHandler、MessageConsumerImpl、MessageConsumerImpl4Client

## 第7章 分布式统一ID生成服务架构设计
#### 7-1 统一ID生成策略_简单生成策略
- 最基础的ID的生成策略：
- java.util.UUID工具类生成
- 雪花算法、数据库sequence序列、自增ID等
- 顺序ID生成方式工具类：KeyUtil

#### 7-2 统一ID生成策略_业务规则策略
- 业务ID生成方式
- 使用带有业务含义的ID生成策略，这种方式也在传统应用系统、特定的场景下非常的好用。比如我们现在有一张商品货架表，这张表的数据维度是这样的，比如是按照城市和区域来划分的
- 比如北京我们按照100000为基本维度数据，100010为北京的一个区域，100020则为另一个区域，以此类推，200000可能是另一个城市，200010则为另一个城市的区域。那么我们在生成货架信息ID的时候，可以按照前六位为城市和区域的方式进行组织，后面可以拼接一个简单的32位UUID字符串
- 前四位为城市维度编码，五六位为区域维度编码，最后两位为货架类型编码

#### 7-3 统一ID生成策略_Zookeeper和Redis的方案在高并发下暴露的问题
- 高并发下的统一ID生成策略服务
- 第一：如何解决ID生成在并发下的重复生成问题
- 第二：如何承载高并发ID生成的性能瓶颈问题。
- 使用Zookeeper的分布式锁实现，不行，Zookeeper写有性能瓶颈
- 使用Redis缓存，利用Redis分布式锁实现，不行，Redis也有性能瓶颈，而且Redis服务调用会失败

#### 7-4 业界主流的分布式高并发ID生成规则方案
- 业界主流的分布式ID生成器的策略是：
- 实现一：提前加载，也就是预加载的机制
- 实现二：单点生成方式
- 分布式ID生成器的策略：预加载机制，其实现核心如下：
- 1：提前加载，也就是预加载的机制
- 2：并发的获取，采用Disruptor框架去提升性能
- 分布式ID生成器的策略：单点生成方式，其实现核心如下：
- 1：固定的一个机器节点来生成一个唯一的ID，好处是能做到全局唯一
- 2：需要相应的业务规则拼接：机器码+时间戳+自增序列

#### 7-5 高并发下分布式ID生成策略经典NTP问题解读
- 为什么要进行拼接自增序列？
- 在高并发的场景下，我们的生成ID请求量非常的巨大，就会暴露出NTP的问题。
- NTP是网络时间协议，它是用来同步网络中各个计算机的时间的协议。
- 举个例子，如果单纯的采用：机器码+时间戳或者机器码+UUID这种组合，在高并发下可能会有订单重复的问题。

#### 7-6 分布式统一ID生成服务系统架构设计讲解
![image.png](https://upload-images.jianshu.io/upload_images/1956963-25f154bb5eb3fe79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## [代码](https://github.com/liuhuiAndroid/disruptor-study)
