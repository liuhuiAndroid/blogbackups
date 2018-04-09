**作用**：可以在不同的线程中互不干扰地存储并提供数据

**使用场景**：Handler创建的时候会采用当前线程的Looper来构造消息循环系统，那么Handler内部如何获取到当前线程的Looper呢？这就要使用ThreadLocal了，ThreadLocal可以在不同的线程中互不干扰地存储并提供数据，通过ThreadLocal可以轻松获取到每个线程的Looper。当然需要注意的是，线程是默认没有Looper的，如果需要使用Handler就必须为线程创建Looper。

**基础知识**：ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，对于其他线程来说则无法获取到数据。一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，就可以考虑采用ThreadLocal。ThreadLocal的另一个使用场景是复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程中的任务过于复杂，可能表现为函数调用栈比较深以及代码入口的多样性，在这种情况下，我们又需要监听器能够贯穿整个线程的执行过程，这个时候可以怎么做呢？其实这时就可以采用ThreadLocal，采用ThreadLocal可以让监听器作为线程内的全局对象而存在，在线程内部只要通过get方法就可以获取到监听器。

**例子**：
```
    private static ThreadLocal<Boolean> mBooleanThreadLocal = new ThreadLocal<>();

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mBooleanThreadLocal.set(true);
        Log.i(TAG, "main : mBooleanThreadLocal = " + mBooleanThreadLocal.get());

        new Thread("Thread#1") {
            @Override
            public void run() {
                mBooleanThreadLocal.set(false);
                Log.i(TAG, "Thread#1 : mBooleanThreadLocal = " + mBooleanThreadLocal.get());
            }
        }.run();

        new Thread("Thread#2") {
            @Override
            public void run() {
                Log.i(TAG, "Thread#2 : mBooleanThreadLocal = " + mBooleanThreadLocal.get());
            }
        }.run();

    }
```
**输出结果**
```
I/Test: main : mBooleanThreadLocal = true
I/Test: Thread#1 : mBooleanThreadLocal = false
I/Test: Thread#2 : mBooleanThreadLocal = false
```
从上面日志可以看出,虽然在不同线程中访问的是同一个ThreadLocal对象，但是它们通过ThreadLocal获取到的值却是不一样的，这就是ThreadLocal的奇妙之处。
ThreadLocal之所以有这么奇妙的效果，是因为不同线程访问同一个ThreadLocal的get方法，ThreadLocal内部会从各自的线程中取出一个数组，然后和再从数组中根据当前ThreadLocal的索引去查找出对应的value值。很显然，不同线程中的数组是不同的，这就是为什么通过ThreadLocal可以在不同的线程中维护一套数据的副本并且彼此互不干扰。

**内部实现**
ThreadLocal是一个泛型类，它的定义为public class ThreadLocal<T>，只要弄清楚ThreadLocal的get和set方法就可以明白它的工作原理。
首先来看ThreadLocal的set方法：
```
// android 23 源码
public void set(T value) {
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values == null) {
            values = initializeValues(currentThread);
        }
        values.put(this, value);
    }
``` 
在上面的set方法中，首先会通过values方法来获取当前线程的ThreadLocal数据，如何获取呢？其实获取的方式也是很简单的，在Thread类的内部有一个成员专门用于存储线程的ThreadLocal的数据：ThreadLocal.Values localValues，因此获取当前线程的ThreadLocal数据就变得异常简单了。如果localValues的值为null，那么就需要对其进行初始化，初始化后再将ThreadLocal的值进行存储。

下面看一个ThreadLocal的值到底是如何在localValues中进行存储的。在localValues内部有一个数组:private Object[] table,ThreadLocal的值就存在在这个table数据中。

下面分析get方法：
```
public T get() {
        // Optimized for the fast path.
        Thread currentThread = Thread.currentThread();
        Values values = values(currentThread);
        if (values != null) {
            Object[] table = values.table;
            int index = hash & values.mask;
            if (this.reference == table[index]) {
                return (T) table[index + 1];
            }
        } else {
            values = initializeValues(currentThread);
        }

        return (T) values.getAfterMiss(this);
    }
```
可以发现，ThreadLocal的get方法的逻辑也比较清晰，它同样是取出当前线程的localValues对象，如果这个对象为null那么久 返回初始值，初始值由ThreadLocal的initializeValues方法来描述。
如果localValues对象不为null，那就读取它的table数据并找出ThreadLocal的reference对象在table数组中的位置，然后table数据中的下一个位置所存储的数据就是ThreadLocal的值。

从ThreadLocal的set和get方法可以看出，它们所操作的对象都是当前线程的localValues对象的table数据，因此在不同的线程中访问同一个ThreadLocal的set和get方法，它们对ThreadLocal所做的读/写操作仅限于各自线程的内部，这就是为什么ThreadLocal可以在多个线程中互不干扰地存储和修改数据。

##代码
[例子代码](https://github.com/liuhuiAndroid/wwh/tree/master/sourceanalysis/src/main/java/com/android/wwh/opensourceprojectanalysis/threadlocal)

##学习于
[Android开发艺术探索](https://book.douban.com/subject/26599538/)
