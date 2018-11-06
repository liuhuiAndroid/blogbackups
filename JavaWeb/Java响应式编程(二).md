# **五、webflux服务端开发讲解**
### 初识SpringWebFlux
和SpringMVC关系
![image.png](https://upload-images.jianshu.io/upload_images/1956963-68baea9713f8eb59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 异步servlet
Servlet 3.0异步处理支持
同步：
```
@WebServlet("/SyncServlet")
public class SyncServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public SyncServlet() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        long t1 = System.currentTimeMillis();
        // 执行业务代码
        doSomeThing(request, response);
        System.out.println("sync use:" + (System.currentTimeMillis() - t1));
    }

    private void doSomeThing(HttpServletRequest request, HttpServletResponse response) throws IOException {
        // 模拟耗时操作
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
        }
        response.getWriter().append("done");
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```
异步：
```
@WebServlet(asyncSupported = true, urlPatterns = {"/AsyncServlet"})
public class AsyncServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public AsyncServlet() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        long t1 = System.currentTimeMillis();
        // 开启异步
        AsyncContext asyncContext = request.startAsync();
        // 执行业务代码
        CompletableFuture.runAsync(() -> doSomeThing(asyncContext, asyncContext.getRequest(), asyncContext.getResponse()));
        System.out.println("async use:" + (System.currentTimeMillis() - t1));
    }

    private void doSomeThing(AsyncContext asyncContext, ServletRequest servletRequest, ServletResponse servletResponse) {
        // 模拟耗时操作
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
        }
        try {
            servletResponse.getWriter().append("done");
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 业务代码处理完毕, 通知结束
        asyncContext.complete();
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```

### HTML 5 服务器发送事件 ( SSE, server-sent event ) 
```
@WebServlet("/SSE")
public class SSE extends HttpServlet {
    private static final long serialVersionUID = 1L;

    public SSE() {
        super();
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("utf-8");
        for (int i = 0; i < 5; i++) {
            // 指定事件标识
            response.getWriter().write("event:me\n");
            // 格式: data: + 数据 + 2个回车
            response.getWriter().write("data:" + i + "\n\n");
            response.getWriter().flush();
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
            }
        }
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }
}
```
```
	<script type="text/javascript">
		// 初始化, 参数为url
		// 依赖H5
		var sse = new EventSource("SSE");

		sse.onmessage = function(e) {
			console.log("message", e.data, e);
		}

		// 监听指定事件, (就不会进入onmessage了)
		sse.addEventListener("me", function(e) {
			console.log("me event", e.data);
			// 如果不关闭,会自动重连
			if (e.data == 3) {
				sse.close();
			}
		});
	</script>
```

### webflux开发
[Mono](https://link.jianshu.com?t=https%3A%2F%2Fprojectreactor.io%2Fdocs%2Fcore%2Frelease%2Freference%2F%23mono), an Asynchronous 0-1 Result
```
     @Test
    public void testMonoBasic() {
        Mono.fromSupplier(() -> "Hello").subscribe(System.out::println);
        Mono.justOrEmpty(Optional.of("Hello")).subscribe(System.out::println);
        Mono.create(sink -> sink.success("Hello")).subscribe(System.out::println);
    }
```
[Flux](https://link.jianshu.com?t=https%3A%2F%2Fprojectreactor.io%2Fdocs%2Fcore%2Frelease%2Freference%2F%23flux), an Asynchronous Sequence of 0-N Items
```
    @Test
    public void testBasic() {
        Flux.just("Hello", "World").subscribe(System.out::println);
        Flux.fromArray(new Integer[]{1, 2, 3}).subscribe(System.out::println);
        Flux.empty().subscribe(System.out::println);
        Flux.range(1, 10).subscribe(System.out::println);
        Flux.interval(Duration.of(10, ChronoUnit.SECONDS)).subscribe(System.out::println);
    }
```

```
@RestController
@Slf4j
public class TestController {

    @GetMapping("/1")
    private String get1() {
        log.info("get1 start");
        String result = createStr();
        log.info("get1 end.");
        return result;
    }

    @GetMapping("/2")
    private Mono<String> get2() {
        log.info("get2 start");
        Mono<String> result = Mono.fromSupplier(() -> createStr());
        log.info("get2 end.");
        return result;
    }

    /**
     * Flux : 返回0-n个元素
     * MediaType.TEXT_EVENT_STREAM表示Content-Type为text/event-stream，即SSE
     */
    @GetMapping(value = "/3", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    private Flux<String> flux() {
        Flux<String> result = Flux
                .fromStream(IntStream.range(1, 5).mapToObj(i -> {
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                    }
                    return "flux data--" + i;
                }));
        return result;
    }

    private String createStr() {
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
        }
        return "some string";
    }

}
```

```
public class ReactorDemo {

	public static void main(String[] args) {
		// reactor spring官方为了支持异步事件驱动而添加的库
		// reactor = jdk8 stream + jdk9 reactive stream
		// Mono 0-1个元素
		// Flux 0-N个元素
		String[] strs = { "1", "2", "3" };

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
		
		// 这里就是jdk8的stream
		Flux.fromArray(strs).map(s -> Integer.parseInt(s))
		// 最终操作
		// 这里就是jdk9的reactive stream
		.subscribe(subscriber);
	}

}
```

### 完整例子
[**mongdb 官网**](https://www.mongodb.com/)
[**Spring Data MongoDB**](https://projects.spring.io/spring-data-mongodb/)

application.properties
```
spring.data.mongodb.uri=mongodb://118.126.111.144:27017/webflux
```
```
@SpringBootApplication
@EnableReactiveMongoRepositories
public class WebfluxApplication {

	public static void main(String[] args) {
		SpringApplication.run(WebfluxApplication.class, args);
	}
	
}
```

### 完整例子- CRUD
```
@RestController
@RequestMapping("/user")
public class UserController {

	private final UserRepository repository;

	// Spring建议”总是在您的bean中使用构造函数建立依赖注入。
	// 使用构造器注入的方法，可以明确成员变量的加载顺序。
	@Autowired
	public UserController(UserRepository repository) {
		this.repository = repository;
	}

	/**
	 * 以数组形式一次性返回数据
	 */
	@GetMapping("/")
	public Flux<User> getAll() {
		return repository.findAll();
	}

	/**
	 * 以SSE形式多次返回数据
	 */
	@GetMapping(value = "/stream/all", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<User> streamGetAll() {
		return repository.findAll();
	}

	/**
	 * 新增数据
	 */
	@PostMapping("/")
	public Mono<User> createUser(@Valid @RequestBody User user) {
		// spring data jpa 里面, 新增和修改都是save. 有id是修改, id为空是新增
		// 根据实际情况是否置空id
		user.setId(null);
		CheckUtil.checkName(user.getName());
		return this.repository.save(user);
	}

	/**
	 * 根据id删除用户 存在的时候返回200, 不存在返回404
	 */
	@DeleteMapping("/{id}")
	public Mono<ResponseEntity<Void>> deleteUser(@PathVariable("id") String id) {
		// deletebyID 没有返回值, 不能判断数据是否存在
		// this.repository.deleteById(id)
		return this.repository.findById(id)
				// 当你要操作数据, 并返回一个Mono 这个时候使用flatMap
				// 如果不操作数据, 只是转换数据, 使用map
				.flatMap(user -> this.repository.delete(user).then(
						Mono.just(new ResponseEntity<Void>(HttpStatus.OK))))
				.defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
	}

	/**
	 * 修改数据 存在的时候返回200 和修改后的数据, 不存在的时候返回404
	 */
	@PutMapping("/{id}")
	public Mono<ResponseEntity<User>> updateUser(@PathVariable("id") String id,
			@Valid @RequestBody User user) {
		CheckUtil.checkName(user.getName());
		return this.repository.findById(id)
				// flatMap 操作数据
				.flatMap(u -> {
					u.setAge(user.getAge());
					u.setName(user.getName());
					return this.repository.save(u);
				})
				// map: 转换数据
				.map(u -> new ResponseEntity<User>(u, HttpStatus.OK))
				.defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
	}

	/**
	 * 根据ID查找用户 存在返回用户信息, 不存在返回404
	 */
	@GetMapping("/{id}")
	public Mono<ResponseEntity<User>> findUserById(@PathVariable("id") String id) {
		return this.repository.findById(id)
				.map(u -> new ResponseEntity<User>(u, HttpStatus.OK))
				.defaultIfEmpty(new ResponseEntity<>(HttpStatus.NOT_FOUND));
	}

	/**
	 * 根据年龄查找用户
	 */
	@GetMapping("/age/{start}/{end}")
	public Flux<User> findByAge(@PathVariable("start") int start,@PathVariable("end") int end) {
		return this.repository.findByAgeBetween(start, end);
	}

	/**
	 * 根据年龄查找用户
	 */
	@GetMapping(value = "/stream/age/{start}/{end}", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<User> streamFindByAge(@PathVariable("start") int start,
			@PathVariable("end") int end) {
		return this.repository.findByAgeBetween(start, end);
	}
	
	/**
	 *  得到20-30用户
	 */
	@GetMapping("/old")
	public Flux<User> oldUser() {
		return this.repository.oldUser();
	}

	/**
	 * 得到20-30用户
	 */
	@GetMapping(value = "/stream/old", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
	public Flux<User> streamOldUser() {
		return this.repository.oldUser();
	}

}
```

### 完整例子-jpa
```
@Repository
public interface UserRepository extends ReactiveMongoRepository<User, String> {

	/**
	 * 根据年龄查找用户
	 */
	Flux<User> findByAgeBetween(int start, int end);
	
	@Query("{'age':{ '$gte': 20, '$lte' : 30}}")
	Flux<User> oldUser();
	
}
```

### 完整例子-参数校验
```
public class CheckUtil {

	private static final String[] INVALID_NAMES = { "admin", "guanliyuan" };

	/**
	 * 校验名字, 不成功抛出校验异常
	 */
	public static void checkName(String value) {
		Stream.of(INVALID_NAMES).filter(name -> name.equalsIgnoreCase(value))
				.findAny()
				.ifPresent(name -> {
					throw new CheckException("name", value);
				});
	}

}
```
```
/**
 * 异常处理切面
 * @ControllerAdvice + @ExceptionHandler 全局处理 Controller 层异常
 */
// @ControllerAdvice 注解定义全局异常处理类
@ControllerAdvice
public class CheckAdvice {

	// @ExceptionHandler 注解声明异常处理方法
	@ExceptionHandler(WebExchangeBindException.class)
	public ResponseEntity<String> handleBindException(WebExchangeBindException e) {
		return new ResponseEntity<String>(toStr(e), HttpStatus.BAD_REQUEST);
	}

	// @ExceptionHandler 注解声明异常处理方法
	@ExceptionHandler(CheckException.class)
	public ResponseEntity<String> handleCheckException(CheckException e) {
		return new ResponseEntity<String>(toStr(e), HttpStatus.BAD_REQUEST);
	}

	private String toStr(CheckException e) {
		return e.getFieldName() + ": 错误的值 " + e.getFieldValue();
	}

	/**
	 * 把校验异常转换为字符串
	 * 
	 * @param ex
	 * @return
	 */
	private String toStr(WebExchangeBindException ex) {
		return ex.getFieldErrors().stream()
				.map(e -> e.getField() + ":" + e.getDefaultMessage())
				.reduce("", (s1, s2) -> s1 + "\n" + s2);
	}

}
```
```
@Data
public class CheckException extends RuntimeException {

	private static final long serialVersionUID = 1L;

	/**
	 * 出错字段的名字
	 */
	private String fieldName;
	
	/**
	 * 出错字段的值
	 */
	private String fieldValue;
	
	public CheckException() {
		super();
	}

	public CheckException(String message, Throwable cause,
			boolean enableSuppression, boolean writableStackTrace) {
		super(message, cause, enableSuppression, writableStackTrace);
	}

	public CheckException(String message, Throwable cause) {
		super(message, cause);
	}

	public CheckException(String message) {
		super(message);
	}

	public CheckException(Throwable cause) {
		super(cause);
	}

	public CheckException(String fieldName, String fieldValue) {
		super();
		this.fieldName = fieldName;
		this.fieldValue = fieldValue;
	}
	
}
```

### RouterFunction模式
代码具体见源码，很简单
思考使用 RouterFunction 的好处？

# **六、webflux客户端声明式restclient框架开发讲解**
### 框架效果介绍
类似Retrofit的网络请求框架。
### 设计思路
![image.png](https://upload-images.jianshu.io/upload_images/1956963-147c3dc73ac48570.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### 框架代码编写
###### 信息提取
###### 处理请求
###### 异常处理
具体详见源码。

# [**响应式编程源码**](https://github.com/liuhuiAndroid/reactive-programming-demo)
