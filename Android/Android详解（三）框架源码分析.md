拆各种轮子，学习各种思想，模仿各种造轮子
[Android 开发中有什么经典的轮子值得自己去实现一遍？](https://www.zhihu.com/question/52530375/answer/131082842?utm_source=qq&utm_medium=social)
[如何正确使用开源项目？](http://mp.weixin.qq.com/s?__biz=MzA4NTQwNDcyMA==&mid=2650661623&idx=1&sn=ab28ac6587e8a5ef1241be7870851355#rd)
===================
- ☐ EventBus实现原理
> -  [EventBus解析合集](http://www.jianshu.com/p/b00f5cd85148)

- ☐ ThreadLocal是什么
>[彻底理解ThreadLocal](http://www.jianshu.com/p/45ecfc95b0c3)

- ☐ handler实现机制（很多细节需要关注：如线程如何建立和退出消息循环等等）
> [Android 异步消息处理机制 让你深入理解 Looper、Handler、Message三者关系](http://blog.csdn.net/lmj623565791/article/details/38377229)
>[Android异步消息处理机制完全解析，带你从源码的角度彻底理解](http://blog.csdn.net/guolin_blog/article/details/9991569)
- ☐ handler发消息给子线程，looper怎么启动
>**Handler总是依附于创建时所在的线程，所以你在子线程A里创建了一个Handler，不管在任何其它地方使用这个Handler去发送消息时，最后肯定会在子线程A里去处理这个消息**

- ☐ Handler用法
>[Android Handler 异步消息处理机制的妙用 创建强大的图片加载类](http://blog.csdn.net/lmj623565791/article/details/38476887)
> [Android IntentService完全解析 当Service遇到Handler](http://blog.csdn.net/lmj623565791/article/details/47143563)

- ☐ handler内存泄露
>[Android App 内存泄露之Handler](http://blog.csdn.net/zhuanglonghai/article/details/38233069)

- ☐ HandlerThread实现
>[Android HandlerThread 完全解析](http://blog.csdn.net/lmj623565791/article/details/47079737)

- ☐ ButterKnife 源码分析
> [ButterKnife 源码分析](http://yuqirong.me/2016/12/18/ButterKnife源码解析/)

- ☐ 解决运行时危险权限方案源码分析
>[AndPermission](https://github.com/yanzhenjie/AndPermission)
>[RxPermissions](https://github.com/tbruyelle/RxPermissions)

- ☐ [dagger2](https://github.com/google/dagger) 学习和源码分析 [ˈdægə(r)] 匕首; 短剑;用剑刺;
> [Android：dagger2让你爱不释手-基础依赖注入框架篇](http://www.jianshu.com/p/cd2c1c9f68d4)
> - Inject主要是用来标注目标类的依赖和依赖的构造函数
> - Component它是一个桥梁，一端是目标类，另一端是目标类所依赖类的实例，它也是注入器（Injector）负责把目标类所依赖类的实例注入到目标类中，同时它也管理Module。
> - Module [ˈmɒdju:l] 和Provides是为解决第三方类库而生的，Module是一个简单工厂模式，Module可以包含创建类实例的方法，这些方法用Provides来标注
>
> [Android：dagger2让你爱不释手-重点概念讲解、融合篇](http://www.jianshu.com/p/1d42d2e6f4a5)
> - 创建类实例级别Module维度要高于Inject维度。
>
> [Android：dagger2让你爱不释手-终结篇](http://www.jianshu.com/p/65737ac39c44)
> - dagger2进行的一次依赖注入的步骤:
> 步骤1：查找Module中是否存在创建该类的方法。
步骤2：若存在创建类方法，查看该方法是否存在参数
    步骤2.1：若存在参数，则按从**步骤1**开始依次初始化每个参数
    步骤2.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束
步骤3：若不存在创建类方法，则查找Inject注解的构造函数，看构造函数是否存在参数
    步骤3.1：若存在参数，则从**步骤1**开始依次初始化每个参数
    步骤3.2：若不存在参数，则直接初始化该类实例，一次依赖注入到此结束
> 
> [Android常用开源工具（1）-Dagger2入门](http://blog.csdn.net/duo2005duo/article/details/50618171)
> - 一个完整的Module必须拥有@Module与@Provides注解 
> - 如果Module只有有参构造器，则必须显式传入Module实例。
> - 添加多个Module的三种方式：
①@Component(modules={××××，×××}) 添加多个modules
②@Module（includes={××××，×××}）
③@Component(dependencies = xxx)
>
>[Android常用开源工具（2）-Dagger2进阶](http://blog.csdn.net/duo2005duo/article/details/50696166)
> - 如果两个Component间有依赖关系，那么它们不能使用相同的Scope
> - 注意事项 坑 ：
 1  - component的inject方法接受父类型参数，而调用时传入的是子类型对象则无法注入
 2 - component关联的modules中不能有重复的provide
 3 - module的provide方法使用了scope,那么component就必须使用同一个注解
 4 - module的provide方法没有使用scope，那么component和module是否加注解都无关紧要，可以通过编译
 5 - component的dependencies与component自身的scope不能相同，即组件之间的scope不同
 6 - singleton的组件不能依赖其他scope的组件，只能其他scope的组件依赖singleton的组件
 7 - 没有scope的component不能依赖有scope的component
 8 - 一个component不能同时又多个scope(Subcomponent除外)
 9 - @Singleton的生命周期依附于component，同一个module provide singleton,不同component也是不一样
>
>
>
>

- ☐ [暂时用 android-wheel](http://code.google.com/p/android-wheel)
> [Android三级联动wheel代码分析（一）](http://www.jianshu.com/p/2044154f8058)
- ☐ **[暂时用 图片选择 PictureSelector](https://github.com/LuckSiege/PictureSelector)**

- ☐ adapter
>
>

- ☐ [timber](https://github.com/JakeWharton/timber)（[logger](https://github.com/orhanobut/logger)）
>[Log最佳实践](http://www.jianshu.com/p/586c27e77e81)
> [Android 从StackTraceElement反观Log库](http://blog.csdn.net/lmj623565791/article/details/52506545)
>

- ☐ Activity中切换不同状态显示页面
>[Android 多状态加载布局的开发 Tips 我觉得写的很好](http://blog.csdn.net/maoruibin9035/article/details/71091390)
>**[StatefulLayout](https://github.com/gturedi/StatefulLayout)**
>**[progress-activity](https://github.com/vlonjatg/progress-activity)**
>

- ☐ 文件下载库
>**[FileDownloader](https://github.com/lingochamp/FileDownloader)**
>

- ☐
