**定义：是一个发布 / 订阅的事件总线。**

### 了解基础知识反射
>[知乎上看到对于反射比较好的回答](https://www.zhihu.com/question/24304289)
>[深入解析Java反射（1） - 基础](http://www.sczyh30.com/posts/Java/java-reflection-1/#%E4%B8%80%E3%80%81%E5%9B%9E%E9%A1%BE%EF%BC%9A%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%8D%E5%B0%84%EF%BC%9F)

### 相关文章推荐
>[Android EventBus实战 没听过你就out了](http://blog.csdn.net/lmj623565791/article/details/40794879)
>在onCreate里面执行EventBus.getDefault().register(this);意思是让EventBus扫描当前类，把所有onEvent开头的方法记录下来，如何记录呢？使用Map，Key为方法的参数类型，Value中包含我们的方法。这样在onCreate执行完成以后，我们的onEventMainThread就已经以键值对的方式被存储到EventBus中了。然后当子线程执行完毕，调用EventBus.getDefault().post(new ItemListEvent(Item.ITEMS))时，EventBus会根据post中实参的类型，去Map中查找对于的方法，于是找到了我们的onEventMainThread，最终调用反射去执行我们的方法。
>
>[Android EventBus源码解析 带你深入理解EventBus](http://blog.csdn.net/lmj623565791/article/details/40920453)
>**其实EventBus就是在内部存储了一堆onEvent开头的方法，然后post的时候，根据post传入的参数，去找到匹配的方法，反射调用之。**
>总结：register会把当前类中匹配的方法，存入一个map，而post会根据实参去map查找进行反射调用。
>**EventBus就是在一个单例内部维持着一个map对象存储了一堆的方法；post无非就是根据参数去查找方法，进行反射调用。**
>
>[Android 框架炼成 教你如何写组件间通信框架EventBus](http://blog.csdn.net/lmj623565791/article/details/41096639)

- EventBus实现线程间的切换
>

- 相关文章
>[Android事件总线（一）EventBus3.0用法全解析](http://liuwangshu.cn/application/eventbus/1-eventbus.html) 
>[Android事件总线（二）EventBus3.0源码解析](http://liuwangshu.cn/application/eventbus/2-eventbus-sourcecode.html)
>[Android事件总线（三）otto用法全解析](http://liuwangshu.cn/application/eventbus/3-otto-sourcecode.html)
>[Android事件总线（四）源码解析otto](http://liuwangshu.cn/application/eventbus/4-otto-sourcecode.html)

**Pass：自己实现了一个精简版本的EventBus**
**[simple-eventbus](https://github.com/liuhuiAndroid/simple-eventbus)**
