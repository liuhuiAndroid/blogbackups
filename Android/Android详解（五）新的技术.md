花五个月时间来系统学习这五个部分
===================
- 注解
>[Java注解（1）-基础](http://blog.csdn.net/duo2005duo/article/details/50505884)
>[Java注解（2）-运行时框架](http://blog.csdn.net/duo2005duo/article/details/50511476)
>[Java注解（3）-源码级框架 // 这个有点蒙](http://blog.csdn.net/duo2005duo/article/details/50541281)
>
>[Java 动态代理机制分析源码](https://github.com/liuhuiAndroid/design-pattern/tree/master/Proxy)
>[Java 动态代理机制分析及扩展，第 1 部分](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/)
>[Java 动态代理机制分析及扩展，第 2 部分](https://www.ibm.com/developerworks/cn/java/j-lo-proxy2/)
>这里主要是对动态代理机制进行了解，还可以参考设计模式里面的知识
>
> [Android 进阶 教你打造 Android 中的 IOC 框架 【ViewInject】 （上）](http://blog.csdn.net/lmj623565791/article/details/39269193)
> [Android 进阶 教你打造 Android 中的 IOC 框架 【ViewInject】 （下）](http://blog.csdn.net/lmj623565791/article/details/39275847)
>以上两篇其实是对应上面的运行时框架
>
>[Annotation实战【自定义AbstractProcessor】](http://www.cnblogs.com/avenwu/p/4173899.html)
>[Android 打造编译时注解解析框架 这只是一个开始](http://blog.csdn.net/lmj623565791/article/details/43452969)
> 这一篇版本比较老，重点看下一篇
>[Android 如何编写基于编译时注解的项目](http://blog.csdn.net/lmj623565791/article/details/51931859)
>以上两篇其实是对应上面的源码级框架

- ☐ 热修复原理
> [Android 热修复 Tinker接入及源码浅析](http://blog.csdn.net/lmj623565791/article/details/54882693)
> [Android 热修复 Tinker 源码分析之DexDiff / DexPatch](http://blog.csdn.net/lmj623565791/article/details/60874334)
> [Android 热修复 Tinker Gradle Plugin解析](http://blog.csdn.net/lmj623565791/article/details/72667669)
>[热修复框架Tinker最完整讲解（01）——集成之路](http://www.jianshu.com/p/ed17f00a3d23)

- ☐ 插件化原理
>[Android插件化：从入门到放弃](http://www.infoq.com/cn/articles/android-plug-ins-from-entry-to-give-up)
>[滴滴插件化方案 VirtualApk 源码解析](http://blog.csdn.net/lmj623565791/article/details/75000580)
>[VirtualAPK插件框架介绍（一）----框架接入](http://www.jianshu.com/p/013510c19391)

- ☐ 热修复技术是怎样实现的，和插件化有什么区别。
>[热修复、热补丁与插件化](http://www.jianshu.com/p/5fe00db03e1d)

- ☐ RxJava2学习
>[给初学者的RxJava2.0教程(一)](http://www.jianshu.com/p/464fa025229e)
>[给初学者的RxJava2.0教程(二)](http://www.jianshu.com/p/8818b98c44e2)
>**CompositeDisposable用于在Activity退出时切断水管**
>[给初学者的RxJava2.0教程(三)](http://www.jianshu.com/p/128e662906af)
>介绍了map、flatMap、concatMap；**flatMap可以处理嵌套请求**
>[给初学者的RxJava2.0教程(四)](http://www.jianshu.com/p/bb58571cdb64)
> 介绍了zip，一个界面需要展示用户的一些信息, 而这些信息分别要从两个服务器接口中获取, 而只有当两个都获取到了之后才能进行展示, 这个时候就可以用Zip了
>[给初学者的RxJava2.0教程(五)](http://www.jianshu.com/p/0f2d6c2387c9)
>了解Backpressure的知识
>[给初学者的RxJava2.0教程(六)](http://www.jianshu.com/p/e4c6d7989356)
>如何解决上下游流速不均衡的问题
>[给初学者的RxJava2.0教程(七)](http://www.jianshu.com/p/9b1304435564)
>了解Flowable的基本知识
>BackpressureStrategy.ERROR:出现上下游流速不均衡的时候直接抛出一个异常
  BackpressureStrategy.BUFFER:不丢弃数据的处理方式
  BackpressureStrategy.DROP:直接把存不下的事件丢弃
  BackpressureStrategy.LATEST:只保留最新的事件
>[给初学者的RxJava2.0教程(八)](http://www.jianshu.com/p/a75ecf461e02)
>[给初学者的RxJava2.0教程(九)](http://www.jianshu.com/p/36e0f7f43a51)
>下游每消费96个事件便会自动触发内部的request()去设置上游的requested的值
>[可能是东半球最全的RxJava2使用场景小结](http://www.jianshu.com/p/9aaccd7bb600)
>[这可能是最好的RxJava 2.x 教程](http://www.jianshu.com/p/0cd258eecf60)

- ☐ Kotlin Kotlin Kotlin
 
- ☐ 待续

- ☐ 甜点
>[你不可不知道的自由泳手臂交叉技术！](http://www.sohu.com/a/154495522_494957)
