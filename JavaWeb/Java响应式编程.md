# **一、介绍**
略.
# **二、函数式编程和lambda表达式**
###函数式编程概念
```
        int[] nums = {33, 44, -55, 66, -77, 88};
        int min = IntStream.of(nums).min().getAsInt();
        System.out.println("min num = " + min);

        // 支持并发操作:ParallelStream
        int min2 = IntStream.of(nums).parallel().min().getAsInt();
        System.out.println("min2 num = " + min2);
```
###为什么要使用函数式编程
```
        // jdk8 lambda
        new Thread(() -> System.out.println("Thread")).start();
```

###lambda初接触
lambda表达式是返回指定接口的对象实例
```
        // lambda表达式
        Interface1 i1 = (i) -> i * 2;
        // i2这种是最常见写法
        Interface1 i2 = i -> i * 2;
        Interface1 i3 = (int i) -> i * 2;
        Interface1 i4 = (int i) -> {
            return i * 2;
        };
```

###jdk8接口新特性
函数接口
```
// 函数接口
@FunctionalInterface
interface Interface1 {
    int doubleNum(int i);

    // 默认实现的方法
    default int add(int x, int y) {
        return x + y;
    }
}

System.out.println(i1.add(3, 7));
System.out.println(i1.doubleNum(7));
```
默认实现的方法
```

@FunctionalInterface
interface Interface2 {
    int doubleNum(int i);

    // 默认实现的方法
    default int add(int x, int y) {
        return x + y;
    }
}

@FunctionalInterface
interface Interface3 extends Interface1, Interface2 {

    @Override
    default int add(int x, int y) {
        return Interface1.super.add(x, y);
    }

}
```

###jdk8重要的函数接口
```
@FunctionalInterface
interface IMoneyFormat {
    String format(int i);
}

class MyMoney {
    private int money;

    public MyMoney(int money) {
        this.money = money;
    }

    public void printMoney1(IMoneyFormat moneyFormat) {
        System.out.println("我的存款：" + moneyFormat.format(money));
    }

    public void printMoney2(Function<Integer, String> integerStringFunction) {
        System.out.println("我的存款：" + integerStringFunction.apply(money));
    }
}

        // jdk8重要的函数接口
        MyMoney myMoney = new MyMoney(999999999);
        myMoney.printMoney1(i -> new DecimalFormat("#,###").format(i));

        Function<Integer, String> moneyFormat = i -> new DecimalFormat("#,###").format(i);
        myMoney.printMoney2(moneyFormat.andThen(s -> "￥" + s));
```

### 函数接口
![image.png](https://upload-images.jianshu.io/upload_images/1956963-35d41c60e600e1f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
		// 断言函数接口
		IntPredicate predicate = i -> i > 0;
		System.out.println(predicate.test(-9));

		// 消费函数接口
		Consumer<String> consumer = s -> System.out.println(s);
		consumer.accept("hello world");
```

### 方法引用
```
class Dog {
    private String name = "大黄";
    private int food = 10;

    public Dog() {
    }

    public Dog(String name) {
        this.name = name;
    }

    public static void bark(Dog dog) {
        System.out.println(dog + "叫了");
    }

    //默认会把当前实例传入到非静态方法，参数名为this，位置是第一个
    public int eat(int num) {
        System.out.println("吃了" + num + "斤狗粮");
        this.food -= num;
        return this.food;
    }

    @Override
    public String toString() {
        return this.name;
    }
}

public class MethodRefrenceDemo {

    public static void main(String[] args) {

        // 方法引用
        Consumer<String> consumer = System.out::println;
        consumer.accept("接受的数据");

        // 静态方法的方法引用
        Consumer<Dog> consumer2 = Dog::bark;
        Dog dog = new Dog();
        consumer2.accept(dog);

        // 非静态方法，使用对象实例的方法引用
        // Function<Integer, Integer> function = dog::eat;
        // UnaryOperator<Integer> function = dog::eat;
        IntUnaryOperator function = dog::eat;
        System.out.println("还剩下" + function.applyAsInt(2) + "斤");

        // 使用类名来方法引用
        BiFunction<Dog, Integer, Integer> eatFunction = Dog::eat;
        System.out.println("还剩下" + eatFunction.apply(dog, 2) + "斤");

        // 构造函数的方法引用
        Supplier<Dog> supplier = Dog::new;
        System.out.println("创建了新对象：" + supplier.get());

        // 带参数的构造函数的方法引用
        Function<String, Dog> function2 = Dog::new;
        System.out.println("创建了新对象：" + function2.apply("旺财"));
    }

}
```
**jclasslib bytecode viewer** 字节码查看工具
**everything** 搜索工具

### 类型推断
```
@FunctionalInterface
interface IMath {
	int add(int x, int y);
}

@FunctionalInterface
interface IMath2 {
	int sub(int x, int y);
}

public class TypeDemo {

	public static void main(String[] args) {
		// 变量类型定义
		IMath lambda = (x, y) -> x + y;
		// 数组里
		IMath[] lambdas = { (x, y) -> x + y };
		// 强转
		Object lambda2 = (IMath) (x, y) -> x + y;
		// 通过返回类型
		IMath createLambda = createLambda();
		TypeDemo demo = new TypeDemo();
		// 当有二义性的时候，使用强转对应的接口解决
		demo.test( (IMath2)(x, y) -> x + y);
	}
	
	public void test(IMath math) {}
	
	public void test(IMath2 math) {}
	
	public static IMath createLambda() {
		return  (x, y) -> x + y;
	}

}
```

### 变量引用
```
		List<String> list = new ArrayList<>();
		Consumer<String> consumer = s -> System.out.println(s + list);
		consumer.accept("1211");
```

### 级联表达式和柯里化
```
/**
 * 级联表达式和柯里化 
 * 柯里化:把多个参数的函数转换为只有一个参数的函数 
 * 柯里化的目的：函数标准化
 * 高阶函数：就是返回函数的函数
 */
public class CurryDemo {

	public static void main(String[] args) {
		// 实现了x+y的级联表达式
		Function<Integer, Function<Integer, Integer>> fun = x -> y -> x + y;
		System.out.println(fun.apply(2).apply(3));

		Function<Integer, Function<Integer, Function<Integer, Integer>>> fun2 = x -> y -> z -> x + y + z;
		System.out.println(fun2.apply(2).apply(3).apply(4));

		int[] nums = { 2, 3, 4 };
		Function f = fun2;
		
		for (int i = 0; i < nums.length; i++) {
			if (f instanceof Function) {
				Object obj = f.apply(nums[i]);
				if (obj instanceof Function) {
					f = (Function) obj;
				} else {
					System.out.println("调用结束：结果为" + obj);
				}
			}
		}
	}
}
```

# **三、Stream流编程**
### Stream流编程-概念
```
public class StreamDemo1 {

	public static void main(String[] args) {
		int[] nums = { 1, 2, 3 };
		// 外部迭代
		int sum = 0;
		for (int i : nums) {
			sum += i;
		}
		System.out.println("结果为：" + sum);

		// 使用stream的内部迭代
		// map就是中间操作（返回stream的操作）
		// sum就是终止操作
		int sum2 = IntStream.of(nums).map(StreamDemo1::doubleNum).sum();
		System.out.println("结果为：" + sum2);

		System.out.println("惰性求值就是终止没有调用的情况下，中间操作不会执行");
		IntStream.of(nums).map(StreamDemo1::doubleNum);
	}

	public static int doubleNum(int i) {
		System.out.println("执行了乘以2");
		return i * 2;
	}

}
```

### 流的创建
![image.png](https://upload-images.jianshu.io/upload_images/1956963-5fe7843929334580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
	public static void main(String[] args) {
		List<String> list = new ArrayList<>();

		// 从集合创建
		list.stream();
		list.parallelStream();

		// 从数组创建
		Arrays.stream(new int[] { 2, 3, 5 });

		// 创建数字流
		IntStream.of(1, 2, 3);
		IntStream.rangeClosed(1, 10);

		// 使用random创建一个无限流
		new Random().ints().limit(10);
		Random random = new Random();

		// 自己产生流
		Stream.generate(() -> random.nextInt()).limit(20);
	}
```

### 流的中间操作
![image.png](https://upload-images.jianshu.io/upload_images/1956963-66dffcb657533c98.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
	public static void main(String[] args) {
		String str = "my name is 01";

		// 把每个单词的长度调用出来
		Stream.of(str.split(" ")).filter(s -> s.length() > 2)
				.map(s -> s.length()).forEach(System.out::println);

		// flatMap A->B属性(是个集合), 最终得到所有的A元素里面的所有B属性集合
		// intStream/longStream 并不是Stream的子类, 所以要进行装箱 boxed
		Stream.of(str.split(" ")).flatMap(s -> s.chars().boxed())
				.forEach(i -> System.out.println((char) i.intValue()));

		// peek 用于debug. 是个中间操作,和 forEach 是终止操作
		System.out.println("--------------peek------------");
		Stream.of(str.split(" ")).peek(System.out::println)
				.forEach(System.out::println);

		// limit 使用, 主要用于无限流
		new Random().ints().filter(i -> i > 100 && i < 1000).limit(10)
				.forEach(System.out::println);
	}
```

### 流的终止操作
![image.png](https://upload-images.jianshu.io/upload_images/1956963-87736b6aa7ad2b22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
	public static void main(String[] args) {
		String str = "my name is 01";

		// 使用并行流
		str.chars().parallel().forEach(i -> System.out.print((char) i));
		System.out.println();

		// 使用 forEachOrdered 保证顺序
		str.chars().parallel().forEachOrdered(i -> System.out.print((char) i));
		System.out.println();

		// 收集到list
		List<String> list = Stream.of(str.split(" "))
				.collect(Collectors.toList());
		System.out.println(list);

		// 使用 reduce 拼接字符串
		Optional<String> letters = Stream.of(str.split(" "))
				.reduce((s1, s2) -> s1 + "|" + s2);
		System.out.println(letters.orElse(""));

		// 带初始化值的reduce
		String reduce = Stream.of(str.split(" ")).reduce("",
				(s1, s2) -> s1 + "|" + s2);
		System.out.println(reduce);

		// 计算所有单词总长度
		Integer length = Stream.of(str.split(" ")).map(s -> s.length())
				.reduce(0, (s1, s2) -> s1 + s2);
		System.out.println(length);

		// max 的使用
		Optional<String> max = Stream.of(str.split(" "))
				.max((s1, s2) -> s1.length() - s2.length());
		System.out.println(max.get());

		// 使用 findFirst 短路操作
		OptionalInt findFirst = new Random().ints().findFirst();
		System.out.println(findFirst.getAsInt());
	}
```

### 并行流
```
public class StreamDemo5 {

    public static void main(String[] args) {
        // 调用parallel 产生一个并行流
		 IntStream.range(1, 100).parallel().peek(StreamDemo5::debug).count();

        // 现在要实现一个这样的效果: 先并行,再串行
        // 多次调用 parallel / sequential, 以最后一次调用为准.
        IntStream.range(1, 100)
                // 调用parallel产生并行流
                .parallel().peek(StreamDemo5::debug)
                // 调用sequential 产生串行流
                .sequential().peek(StreamDemo5::debug2)
                .count();

        // 并行流使用的线程池: ForkJoinPool.commonPool
        // 默认的线程数是 当前机器的cpu个数
        // 使用这个属性可以修改默认的线程数
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "5");
        IntStream.range(1, 100).parallel().peek(StreamDemo5::debug).count();

        // 使用自己的线程池, 不使用默认线程池, 防止任务被阻塞
        // 线程名字 : ForkJoinPool-1
		ForkJoinPool pool = new ForkJoinPool(20);
        pool.submit(() -> IntStream.range(1, 100).parallel()
                .peek(StreamDemo5::debug).count());
        pool.shutdown();
        synchronized (pool) {
            try {
                pool.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void debug(int i) {
        System.out.println(Thread.currentThread().getName() + " debug " + i);
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void debug2(int i) {
        System.err.println("debug2 " + i);
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 收集器
```
	public static void main(String[] args) {
		// 测试数据
		List<Student> students = Arrays.asList(
				new Student("小明", 10, Gender.MALE, Grade.ONE),
				new Student("大明", 9, Gender.MALE, Grade.THREE),
				new Student("小白", 8, Gender.FEMALE, Grade.TWO),
				new Student("小黑", 13, Gender.FEMALE, Grade.FOUR),
				new Student("小红", 7, Gender.FEMALE, Grade.THREE),
				new Student("小黄", 13, Gender.MALE, Grade.ONE),
				new Student("小青", 13, Gender.FEMALE, Grade.THREE),
				new Student("小紫", 9, Gender.FEMALE, Grade.TWO),
				new Student("小王", 6, Gender.MALE, Grade.ONE),
				new Student("小李", 6, Gender.MALE, Grade.ONE),
				new Student("小马", 14, Gender.FEMALE, Grade.FOUR),
				new Student("小刘", 13, Gender.MALE, Grade.FOUR));

		// 得到所有学生的年龄列表
		// s -> s.getAge() --> Student::getAge , 不会多生成一个类似 lambda$0这样的函数
		Set<Integer> ages = students.stream().map(Student::getAge)
				.collect(Collectors.toCollection(TreeSet::new));
		System.out.println("所有学生的年龄:" + ages);

		// 统计汇总信息
		IntSummaryStatistics agesSummaryStatistics = students.stream()
				.collect(Collectors.summarizingInt(Student::getAge));
		System.out.println("年龄汇总信息:" + agesSummaryStatistics);

		// 分块
		Map<Boolean, List<Student>> genders = students.stream().collect(
				Collectors.partitioningBy(s -> s.getGender() == Gender.MALE));
		// System.out.println("男女学生列表:" + genders);
		MapUtils.verbosePrint(System.out, "男女学生列表", genders);

		// 分组
		Map<Grade, List<Student>> grades = students.stream()
				.collect(Collectors.groupingBy(Student::getGrade));
		MapUtils.verbosePrint(System.out, "学生班级列表", grades);

		// 得到所有班级学生的个数
		Map<Grade, Long> gradesCount = students.stream().collect(Collectors
				.groupingBy(Student::getGrade, Collectors.counting()));
		MapUtils.verbosePrint(System.out, "班级学生个数列表", gradesCount);
	}
```

### Stream运行机制
```
/**
 * 验证stream运行机制
 * 1. 所有操作是链式调用, 一个元素只迭代一次 
 * 2. 每一个中间操作返回一个新的流. 流里面有一个属性sourceStage 指向同一个 地方,就是Head 
 * 3. Head->nextStage->nextStage->... -> null
 * 4. 有状态操作会把无状态操作阶段,单独处理
 * 5. 并行环境下, 有状态的中间操作不一定能并行操作.
 * 6. parallel/ sequetial 这2个操作也是中间操作(也是返回stream)
 * 但是他们不创建流, 他们只修改 Head的并行标志
 */
public class RunStream {

	public static void main(String[] args) {
		Random random = new Random();
		// 随机产生数据
		Stream<Integer> stream = Stream.generate(() -> random.nextInt())
				// 产生500个 ( 无限流需要短路操作. )
				.limit(500)
				// 第1个无状态操作
				.peek(s -> print("peek: " + s))
				// 第2个无状态操作
				.filter(s -> {
					print("filter: " + s);
					return s > 1000000;
				})
				// 有状态操作
				.sorted((i1, i2) -> {
					print("排序: " + i1 + ", " + i2);
					return i1.compareTo(i2);
				})
				// 又一个无状态操作
				.peek(s -> {
					print("peek2: " + s);
				}).parallel();

		// 终止操作
		stream.count();
	}

	/**
	 * 打印日志并sleep 5 毫秒
	 * @param s
	 */
	public static void print(String s) {
		// 带线程名(测试并行情况)
		System.out.println(Thread.currentThread().getName() + " > " + s);
		try {
			TimeUnit.MILLISECONDS.sleep(3);
		} catch (InterruptedException e) {
		}
	}
}
```

# **四、reactive stream 响应式流**
### 初识Reactive Stream
背压：发布者和订阅者之间交流

### Reactive Stream主要接口
Flow的实现类：Publisher、 Subscriber、 Subscription、 Processor
![图片.png](https://upload-images.jianshu.io/upload_images/1956963-0f200f889ce509b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 完整实例
```
public class FlowDemo {

	public static void main(String[] args) throws Exception {
		// 1. 定义发布者, 发布的数据类型是 Integer
		// 直接使用jdk自带的SubmissionPublisher, 它实现了 Publisher 接口
		SubmissionPublisher<Integer> publiser = new SubmissionPublisher<Integer>();

		// 2. 定义订阅者
		Subscriber<Integer> subscriber = new Subscriber<Integer>() {

			private Subscription subscription;

			@Override
			public void onSubscribe(Subscription subscription) {
				// 保存订阅关系, 需要用它来给发布者响应
				this.subscription = subscription;

				// 请求一个数据
				this.subscription.request(1);
			}

			@Override
			public void onNext(Integer item) {
				// 接受到一个数据, 处理
				System.out.println("接受到数据: " + item);

				try {
					TimeUnit.SECONDS.sleep(3);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				
				// 处理完调用request再请求一个数据
				this.subscription.request(1);

				// 或者 已经达到了目标, 调用cancel告诉发布者不再接受数据了
				// this.subscription.cancel();
			}

			@Override
			public void onError(Throwable throwable) {
				// 出现了异常(例如处理数据的时候产生了异常)
				throwable.printStackTrace();

				// 我们可以告诉发布者, 后面不接受数据了
				this.subscription.cancel();
			}

			@Override
			public void onComplete() {
				// 全部数据处理完了(发布者关闭了)
				System.out.println("处理完了!");
			}

		};

		// 3. 发布者和订阅者 建立订阅关系
		publiser.subscribe(subscriber);

		// 4. 生产数据, 并发布
		// 这里忽略数据生产过程
		for (int i = 0; i < 1000; i++) {
			System.out.println("生成数据:" + i);
			// submit是个block方法
			publiser.submit(i);
		}

		// 5. 结束后 关闭发布者
		// 正式环境 应该放 finally 或者使用 try-resouce 确保关闭
		publiser.close();

		// 主线程延迟停止, 否则数据没有消费就退出
		Thread.currentThread().join(1000);
		// debug的时候, 下面这行需要有断点
		// 否则主线程结束无法debug
		System.out.println();
	}
}
```

### 运行机制
```
/**
 * 带 process 的 flow demo
 * Processor, 需要继承SubmissionPublisher并实现Processor接口
 * 输入源数据 integer, 过滤掉小于0的, 然后转换成字符串发布出去
 */
class MyProcessor extends SubmissionPublisher<String> implements Processor<Integer, String> {

    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription subscription) {
        // 保存订阅关系, 需要用它来给发布者响应
        this.subscription = subscription;

        // 请求一个数据
        this.subscription.request(1);
    }

    @Override
    public void onNext(Integer item) {
        // 接受到一个数据, 处理
        System.out.println("处理器接受到数据: " + item);

        // 过滤掉小于0的, 然后发布出去
        if (item > 0) {
            this.submit("转换后的数据:" + item);
        }

        // 处理完调用request再请求一个数据
        this.subscription.request(1);

        // 或者 已经达到了目标, 调用cancel告诉发布者不再接受数据了
        // this.subscription.cancel();
    }

    @Override
    public void onError(Throwable throwable) {
        // 出现了异常(例如处理数据的时候产生了异常)
        throwable.printStackTrace();

        // 我们可以告诉发布者, 后面不接受数据了
        this.subscription.cancel();
    }

    @Override
    public void onComplete() {
        // 全部数据处理完了(发布者关闭了)
        System.out.println("处理器处理完了!");
        // 关闭发布者
        this.close();
    }
}

public class FlowDemo2 {

    public static void main(String[] args) throws Exception {
        // 1. 定义发布者, 发布的数据类型是 Integer
        // 直接使用jdk自带的SubmissionPublisher
        SubmissionPublisher<Integer> publiser = new SubmissionPublisher<Integer>();

        // 2. 定义处理器, 对数据进行过滤, 并转换为String类型
        MyProcessor processor = new MyProcessor();

        // 3. 发布者 和 处理器 建立订阅关系
        publiser.subscribe(processor);

        // 4. 定义最终订阅者, 消费 String 类型数据
        Subscriber<String> subscriber = new Subscriber<String>() {

            private Subscription subscription;

            @Override
            public void onSubscribe(Subscription subscription) {
                // 保存订阅关系, 需要用它来给发布者响应
                this.subscription = subscription;

                // 请求一个数据
                this.subscription.request(1);
            }

            @Override
            public void onNext(String item) {
                // 接受到一个数据, 处理
                System.out.println("接受到数据: " + item);

                // 处理完调用request再请求一个数据
                this.subscription.request(1);

                // 或者 已经达到了目标, 调用cancel告诉发布者不再接受数据了
                // this.subscription.cancel();
            }

            @Override
            public void onError(Throwable throwable) {
                // 出现了异常(例如处理数据的时候产生了异常)
                throwable.printStackTrace();

                // 我们可以告诉发布者, 后面不接受数据了
                this.subscription.cancel();
            }

            @Override
            public void onComplete() {
                // 全部数据处理完了(发布者关闭了)
                System.out.println("处理完了!");
            }

        };

        // 5. 处理器 和 最终订阅者 建立订阅关系
        processor.subscribe(subscriber);

        // 6. 生产数据, 并发布
        // 这里忽略数据生产过程
        publiser.submit(-111);
        publiser.submit(111);

        // 7. 结束后 关闭发布者
        // 正式环境 应该放 finally 或者使用 try-resouce 确保关闭
        publiser.close();

        // 主线程延迟停止, 否则数据没有消费就退出
        Thread.currentThread().join(1000);
    }
}
```
