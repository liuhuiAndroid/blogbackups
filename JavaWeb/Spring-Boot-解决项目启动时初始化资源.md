## 实现
在我们实际工作中，总会遇到这样需求，在项目启动的时候需要做一些初始化的操作，比如初始化线程池等。
CommandLineRunner 接口的 Component 会在所有 Spring Beans 都初始化之后，SpringApplication.run() 之前执行，非常适合在应用程序启动之初进行一些数据初始化的工作。
```
@SpringBootApplication(scanBasePackages = "com.lh.diveinspringboot")
public class SpringBootWebMvcBootstrap {

    public static void main(String[] args) {
        System.out.println("The service to start.");
        SpringApplication.run(SpringBootWebMvcBootstrap.class);
        System.out.println("The service has started.");
    }

}
```
```
@Component
@Order(1)
public class OrderRunner1 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner1 start to initialize ...");
    }
}
```
```
@Component
@Order(2)
public class OrderRunner2 implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("The OrderRunner2 start to initialize ...");
    }
}
```
测试结果：
```
The service to start.

 /           /
        _ _    ___  ___            ___
|      | | )| |___ |___      \   )|   )|   )
|      |  / |  __/  __/       \_/ |__/ |__/
                               /
 ...

The OrderRunner1 start to initialize ...
The OrderRunner2 start to initialize ...
The service has started.
```
## 原理
SpringApplication#callRunners
```
	private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
		// 1.获取所有实现ApplicationRunner接口的类
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
		// 2.获取所有实现CommandLineRunner接口的类
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
		// 3.根据其@Order进行排序
		AnnotationAwareOrderComparator.sort(runners);
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
				// 4.调用ApplicationRunner其run方法
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				// 5.调用CommandLineRunner其run方法
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}
```





