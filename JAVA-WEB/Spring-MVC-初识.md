###一、SpringMVC介绍
Spring Web MVC 框架是一个轻量级Web框架，是Spring的一个子框架，天生与Spring框架集成，拥有Spring的特性。
处理流程：
![image.png](https://upload-images.jianshu.io/upload_images/1956963-5d0cc0944640e394.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1956963-62e1c755d0250439.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###二、SpringMVC架构讲解
######1. 框架结构
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a9dab7f30611a0a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
######2. 组件说明
**处理器映射器**、**处理器适配器**、**视图解析器**称为SpringMVC的三大组件。
* DispatcherServlet：前端控制器
用户请求到达前端控制器，它就相当于MVC模式中的C，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。
* **HandlerMapping**：处理器映射器
HandlerMapping负责根据用户请求url找到Handler即处理器，SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
* **HandlAdapter**：处理器适配器
通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
* **ViewResolver**：视图解析器
ViewResolver负责将处理结果生成View视图，ViewResolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。

######3. 默认加载的组件
SpringMVC框架中的DispatcherServlet.properties配置文件已经配置了默认的组件了。
```
org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver
```
######4. 组件扫描器
使用组件扫描器省去在spring容器配置每个Controller类的繁琐。
使用<context:component-scan>自动扫描标记@Controller的控制器类，在springmvc.xml配置文件中配置如下:
```
	<!-- 配置controller扫描包 -->
	<context:component-scan base-package="com.study.springmvc.controller" />
```
######5. 处理器映射器
**注解式处理器映射器**，对类中标记了@ResquestMapping的方法进行映射。根据@ResquestMapping定义的url匹配@ResquestMapping标记的方法，匹配成功返回HandlerMethod对象给前端控制器。
HandlerMethod对象中封装url对应的方法Method。 
推荐使用RequestMappingHandlerMapping完成注解式处理器映射。
在springmvc.xml配置文件中配置如下:
```
	<!-- 配置处理器映射器 -->
	<bean
	class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
```
注解描述：
@RequestMapping：定义请求url到处理器功能方法的映射

######6. 处理器适配器
**注解式处理器适配器**，对标记@ResquestMapping的方法进行适配。
推荐使用RequestMappingHandlerAdapter完成注解式处理器适配。
在springmvc.xml配置文件中配置如下:
```
	<!-- 配置处理器适配器 -->
	<bean
	class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
```

######7. 注解驱动
直接配置处理器映射器和处理器适配器比较麻烦，可以使用注解驱动来加载。
SpringMVC使用<mvc:annotation-driven>自动加载RequestMappingHandlerMapping和RequestMappingHandlerAdapter
可以在springmvc.xml配置文件中使用<mvc:annotation-driven>替代注解处理器和适配器的配置。
```
	<!-- 注解驱动 -->
	<mvc:annotation-driven />
```

######8. 视图解析器
视图解析器使用SpringMVC框架默认的InternalResourceViewResolver，这个视图解析器支持JSP视图解析。
在springmvc.xml配置文件中配置如下：
```
        <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        	<property name="prefix" value="/WEB-INF/jsp/"/>
        	<property name="suffix" value=".jsp"/>
        </bean>
```
逻辑视图名需要在Controller中返回ModelAndView指定，
比如逻辑视图名为itemlist，则最终返回的jsp视图地址："WEB-INF/jsp/itemlist.jsp"
最终jsp物理地址：前缀+逻辑视图名+后缀

###三、参数绑定
######1. SpringMVC默认支持的类型
http://127.0.0.1:8080/springmvc-demo/itemEdit.action?id=1
想从请求的参数中把请求的id取出来。
id包含在Request对象中。可以从Request对象中取id。
想获得Request对象只需要在Controller方法的形参中添加一个参数即可。Springmvc框架会自动把Request对象传递给方法。
代码如下：
```
@RequestMapping("/itemEdit")
public ModelAndView queryItemById(HttpServletRequest request) {
	// 从request中获取请求参数
	String strId = request.getParameter("id");
	Integer id = Integer.valueOf(strId);
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);
	// 把结果传递给页面
	ModelAndView modelAndView = new ModelAndView();
	// 把商品数据放在模型中
	modelAndView.addObject("item", item);
	// 设置逻辑视图
	modelAndView.setViewName("itemEdit");
	return modelAndView;
}
```
* **默认支持的参数类型**包括**HttpServletRequest**、**HttpServletResponse**和**HttpSession**
* **Model**
除了ModelAndView以外，还可以使用Model来向页面传递数据，Model是一个接口，在参数里直接声明Model即可。
如果使用Model则可以不使用ModelAndView对象，Model对象可以向页面传递数据，View对象则可以使用String返回值替代。
不管是Model还是ModelAndView，其本质都是使用Request对象向jsp传递数据。
代码实现：
```
@RequestMapping("/itemEdit")
public String queryItemById(HttpServletRequest request, Model model) {
	// 从request中获取请求参数
	String strId = request.getParameter("id");
	Integer id = Integer.valueOf(strId);
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);
	// 把商品数据放在模型中
	model.addAttribute("item", item);
	return "itemEdit";
}
```
* **ModelMap**
ModelMap是Model接口的实现类，也可以通过ModelMap向页面传递数据。
使用Model和ModelMap的效果一样，如果直接使用Model，SpringMVC会实例化ModelMap。
代码实现：
```
@RequestMapping("/itemEdit")
public String queryItemById(HttpServletRequest request, ModelMap model) {
	// 从request中获取请求参数
	String strId = request.getParameter("id");
	Integer id = Integer.valueOf(strId);
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);
	// 把商品数据放在模型中
	model.addAttribute("item", item);
	return "itemEdit";
}
```
######2. 绑定简单数据类型
当请求的参数名称和处理器形参**名称一致**时会将请求参数与形参进行绑定。
这样，从Request取参数的方法就可以进一步简化。
```
@RequestMapping("/itemEdit")
public String queryItemById(int id, ModelMap model) {
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);
	// 把商品数据放在模型中
	model.addAttribute("item", item);
	return "itemEdit";
}
```
* 支持的数据类型
参数类型推荐使用**包装数据类型**，因为基础数据类型不可以为null
整形：Integer、int
字符串：String
单精度：Float、float
双精度：Double、double
布尔型：Boolean、boolean
对于布尔类型的参数，请求的参数值为true或false。或者1或0
* **@RequestParam**
使用@RequestParam常用于处理简单类型的绑定。
**value**：入参的请求参数名字
**required**：是否必须，默认是true，表示请求中一定要有相应的参数，否则将报错
**defaultValue**：默认值，表示如果请求中没有同名参数时的默认值
```
@RequestMapping("/itemEdit")
public String queryItemById(@RequestParam(value = "itemId", required = true, defaultValue = "1") Integer id,ModelMap modelMap) {
	// 根据id查询商品数据
	Item item = this.itemService.queryItemById(id);
	// 把商品数据放在模型中
	modelMap.addAttribute("item", item);
	return "itemEdit";
}
```
######3. 绑定POJO类型
如果提交的参数很多，或者提交的表单中的内容很多的时候,可以使用简单类型接收数据,也可以使用POJO接收数据。
要求：POJO对象中的属性名和表单中input的name属性一致。

######4. 绑定POJO包装类型
```
查询条件：
<table width="100%" border=1>
	<tr>
		<td>商品id<input type="text" name="item.id"/></td>
		<td>商品名称<input type="text" name="item.name"/></td>
		<td><input type="submit" value="查询"/></td>
	</tr>
</table>
```
```
	public class QueryVo {
		private Item item;
	}
```
```
	// 绑定包装数据类型
	@RequestMapping("/queryItem")
	public String queryItem(QueryVo queryVo) {
		System.out.println(queryVo.getItem().getId());
		System.out.println(queryVo.getItem().getName());
		return "success";
	}
```
######5. 自定义参数绑定
比如日期数据有很多种格式,SpringMVC没办法把字符串转换成日期类型。所以需要自定义参数绑定。
```
	<tr>
		<td>商品生产日期</td>
		<td><input type="text" name="items.createtime"
			value="<fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/>" /></td>
	</tr>
```
自定义Converter
```
//Converter<S, T>
//S:source,需要转换的源的类型
//T:target,需要转换的目标类型
public class DateConveter implements Converter<String, Date>{
	public Date convert(String source) {
		try {
			if(source != null){
				DateFormat df = new SimpleDateFormat("yyyy:MM:dd HH:mm:ss");
				return df.parse(source);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
}
```
配置Converter
```
        <!-- 注解驱动 -->
        <mvc:annotation-driven conversion-service="conversionServiceFactoryBean"/>
        
        <!-- 配置Conveter转换器  转换工厂 （日期、去掉前后空格）。。 -->
        <bean id="conversionServiceFactoryBean" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        	<!-- 配置 多个转换器-->
        	<property name="converters">
        		<list>
        			<bean class="com.study.springmvc.conversion.DateConveter"/>
        		</list>
        	</property>
        </bean>
```

###四、高级参数绑定
######1. 数组类型的参数绑定
Controller方法中可以用Integer[]接收，或者POJO的Integer[]属性接收。
```
public class QueryVo {
	Integer[] ids;
}
```
```
	@RequestMapping("updates")
	public String queryItem(QueryVo queryVo, Integer[] ids) {
		System.out.println(queryVo.getItems().getId());
		System.out.println(queryVo.getItems().getName());
		System.out.println(queryVo.getIds().length);
		System.out.println(ids.length);
		return "success";
	}
```
######2. List类型的绑定
接收List类型的数据必须是POJO的属性，如果方法的形参为ArrayList类型无法正确接收到数据。
```
public class QueryVo {
	private List<Items> itemsList;
}
```
###五、@RequestMapping注解的使用
通过@RequestMapping注解可以定义不同的处理器映射规则。
* URL路径映射
@RequestMapping(value="item")或@RequestMapping("/item"）
value的值是数组，可以将多个url映射到同一个方法
```
@RequestMapping(value = { "itemList", "itemListAll" })
public ModelAndView queryItemList() {
	List<Item> list = this.itemService.queryItemList();
	ModelAndView mv = new ModelAndView("itemList");
	mv.addObject("itemList", list);
	return mv;
}
```
* 添加在类上面
在class上添加@RequestMapping(url)指定通用请求前缀， 限制此类下的所有方法请求url必须以请求前缀开头，可以使用此方法对url进行分类管理
```
@Controller
@RequestMapping("my")
public class ItemController {
}
```

* 请求方法限定
限定GET方法
```
@RequestMapping(value = "/login.action",method = RequestMethod.GET)
```
限定POST方法
```
@RequestMapping(value = "/login.action",method = RequestMethod.POST)
```
GET和POST都可以
```
@RequestMapping(value = "/login.action",method = {RequestMethod.GET,RequestMethod.POST} )
```
###六、Controller方法返回值
* 返回ModelAndView
* 返回void
在Controller方法形参上可以定义request和response，使用request或response指定响应结果：
1 . 使用request转发页面，如下：
request.getRequestDispatcher("页面路径").forward(request, response);
request.getRequestDispatcher("/WEB-INF/jsp/success.jsp").forward(request, response);
2 . 可以通过response页面重定向：
response.sendRedirect("url")
response.sendRedirect("/springmvc-web2/itemEdit.action");
3 . 可以通过response指定响应结果，例如响应json数据如下：
response.getWriter().print("{\"abc\":123}");
```
	@RequestMapping(value = {"itemlist.action"})
	public void itemList(Model model,HttpServletRequest request,HttpServletResponse response) throws Exception{
		//request.getRequestDispatcher("/WEB-INF/jsp/success.jsp").forward(request,response);
		response.sendRedirect("/springmvc-mybatis/itemEdit.action");
		//response.getWriter().print("{\"abc\":123}");
	}
```
* 返回字符串
1 . 逻辑视图名
Controller方法返回字符串可以指定逻辑视图名，通过视图解析器解析为物理视图地址。
```
//指定逻辑视图名，经过视图解析器解析为jsp物理路径：/WEB-INF/jsp/itemList.jsp
return "itemList";
```
2 . Redirect重定向
Controller方法返回字符串可以重定向到一个url地址。
如下商品修改提交后重定向到商品编辑页面。
```
@RequestMapping("updateItem")
public String updateItemById(Item item) {
	// 更新商品
	this.itemService.updateItemById(item);

	// 修改商品成功后，重定向到商品编辑页面
	// 重定向后浏览器地址栏变更为重定向的地址，
	// 重定向相当于执行了新的request和response，所以之前的请求参数都会丢失
	// 如果要指定请求参数，需要在重定向的url后面添加 ?itemId=1 这样的请求参数
	return "redirect:/itemEdit.action?itemId=" + item.getId();
}
```
3 . Forward转发
Controller方法执行后继续执行另一个Controller方法。
如商品修改提交后转向到商品修改页面，修改商品的id参数可以带到商品修改方法中。
```
@RequestMapping("updateItem")
public String updateItemById(Item item) {
	// 更新商品
	this.itemService.updateItemById(item);

	// 修改商品成功后，继续执行另一个方法
	// 使用转发的方式实现。转发后浏览器地址栏还是原来的请求地址，
	// 转发并没有执行新的request和response，所以之前的请求参数都存在
	// 结果转发到editItem.action，request可以带过去
	return "forward:/itemEdit.action";
}
```

###七、SpringMVC中异常处理
SpringMVC在处理请求过程中出现异常信息交由异常处理器进行处理，自定义异常处理器可以实现一个系统的异常处理逻辑。
* 自定义异常类
为了区别不同的异常,通常根据异常类型进行区分，这里我们创建一个自定义系统异常。
```
public class MyException extends Exception{
	private String msg;
}
```
* 自定义异常处理器
```
public class CustomExceptionResolver implements HandlerExceptionResolver{

	private static final Logger logger = LoggerFactory.getLogger(CustomExceptionResolver.class); 

	public ModelAndView resolveException(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2,
			Exception exception) {
		// TODO 加入日志
		logger.error("系统发生异常", ex);

		ModelAndView mav = new ModelAndView();
		//判断异常为类型
		if(exception instanceof MessageException){
			//预期异常
			MessageException me = (MessageException)e;
			mav.addObject("error", me.getMsg());
		}else{
			mav.addObject("error", "未知异常");
		}
		mav.setViewName("error");
		return mav;
	}

}
```
log4j.properties
```
log4j.rootLogger=INFO,A3,STDOUT

log4j.appender.STDOUT=org.apache.log4j.ConsoleAppender
log4j.appender.STDOUT.layout=org.apache.log4j.PatternLayout
log4j.appender.STDOUT.layout.ConversionPattern=[%p] [%l] %10.10c - %m%n

log4j.appender.A3=org.apache.log4j.RollingFileAppender
log4j.appender.A3.file=logs/server.log
log4j.appender.A3.MaxFileSize=1024KB
log4j.appender.A3.MaxBackupIndex=10
log4j.appender.A3.layout=org.apache.log4j.PatternLayout
log4j.appender.A3.layout.ConversionPattern=\n\n[%-5p] %d{yyyy-MM-dd HH\:mm\:ss,SSS} method\:%l%n%m%n
```
```
			<dependency>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
				<version>${slf4j.version}</version>
			</dependency>
```
* 异常处理器配置
在springmvc.xml中添加：
```
        <!-- Springmvc的异常处理器 -->
	<bean class="com.study.springmvc.exception.CustomExceptionResolver"/>
```
###八、图片上传处理
* tomcat配置图片虚拟目录
在tomcat下conf/server.xml中添加：
```
<Context docBase="D:\upload\" path="/pic" reloadable="false"/>
```
访问http://localhost:8080/pic即可访问D:\develop下的图片
* 图片上传所需要的jar
commons-fileupload-1.2.2.jar
commons-io-2.4.jar
* 上传解析器
在springmvc.xml中配置文件上传解析器
```
        <!-- 上传图片配置实现类 -->
        <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        	<!-- 上传图片的大小   B   5M  1*1024*1024*5-->
        	<property name="maxUploadSize" value="5000000"/>
        </bean>
```
* 页面修改
```
			<tr>
				<td>商品图片</td>
				<td>
					<c:if test="${item.pic !=null}">
						<img src="/pic/${item.pic}" width=100 height=100/>
						<br/>
					</c:if>
					<input type="file"  name="pictureFile"/> 
				</td>
			</tr>
```
设置表单可以进行文件上传
```
<form id="itemForm"	action="" method="post" enctype="multipart/form-data">
</form>
```
* 上传逻辑
```
@RequestMapping("updateItem")
public String updateItemById(Item item, MultipartFile pictureFile) throws Exception {
	// 图片上传
	// 设置图片名称，不能重复，可以使用uuid
	String picName = UUID.randomUUID().toString();
	// 获取文件名
	String oriName = pictureFile.getOriginalFilename();
	// 获取图片后缀
	String extName = oriName.substring(oriName.lastIndexOf("."));
	// 开始上传
	pictureFile.transferTo(new File("D:/upload/" + picName + extName));
	// 设置图片名到商品中
	item.setPic(picName + extName);
	// 更新商品
	this.itemService.updateItemById(item);
	return "forward:/itemEdit.action";
}
```
###九、Json数据交互
* **@RequestBody**
@RequestBody注解用于读取http请求的内容(字符串)，通过springmvc提供的HttpMessageConverter接口将读到的内容（json数据）转换为java对象并绑定到Controller方法的参数上。
* **@ResponseBody**
@ResponseBody注解用于将Controller的方法返回的对象，通过springmvc提供的HttpMessageConverter接口转换为指定格式的数据如：json,xml等，通过Response响应给客户端
* **实现请求Json，响应Json**
1 . 加入json类库的jar包，比如Gson或Jackson
jackson-annotations-2.4.0.jar
jackson-core-2.4.2.jar
jackson-databind-2.4.2.jar
2 .Controller编写
```
@RequestMapping("json")
public @ResponseBody Item testJson(@RequestBody Item item) {
	return item;
}
```
3 . 测试
![image.png](https://upload-images.jianshu.io/upload_images/1956963-a2ae68cfb7d16c90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###十、SpringMVC实现RESTful
Restful就是一个资源定位及资源操作的风格。
传统方式操作资源
http://127.0.0.1/item/queryItem.action?id=1	查询,GET
http://127.0.0.1/item/saveItem.action			新增,POST
http://127.0.0.1/item/updateItem.action		更新,POST
http://127.0.0.1/item/deleteItem.action?id=1	删除,GET或POST
使用RESTful操作资源
http://127.0.0.1/item/1		查询,GET
http://127.0.0.1/item		新增,POST
http://127.0.0.1/item		更新,PUT
http://127.0.0.1/item/1		删除,DELETE

* 使用注解@RequestMapping("item/{xxx}")声明请求的url，{xxx}叫做占位符，请求的url可以是“item /1”或“item/2”
* 使用(@PathVariable() Integer id)获取url上的数据
* @PathVariable是获取url上数据的。@RequestParam获取请求参数的
```
@RequestMapping("item/{id}")
@ResponseBody
public Item queryItemById(@PathVariable() Integer id) {
	Item item = this.itemService.queryItemById(id);
	return item;
}
```
如果加上@ResponseBody注解，就不会走视图解析器，不会返回页面，目前返回的json数据。如果不加，就走视图解析器，返回页面
###十一、拦截器
Spring MVC 的处理器拦截器类似于Servlet 开发中的过滤器Filter，用于对处理器进行预处理和后处理。
* 拦截器定义
实现HandlerInterceptor接口
```
public class HandlerInterceptor1 implements HandlerInterceptor{

	// Controller执行前调用此方法
	// 返回true表示继续执行，返回false中止执行
	// 这里可以加入登录校验、权限拦截等
	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
		System.out.println("HandlerInterceptor1....preHandle");
		// 设置为true，测试使用
		return true;
	}
	
	// Controller执行后但未返回视图前调用此方法
	// 这里可在返回用户前对模型数据进行加工处理，比如这里加入公用信息以便页面显示
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3)
			throws Exception {
		System.out.println("HandlerInterceptor1....postHandle");
	}
	
	// Controller执行后且视图返回后调用此方法
	// 这里可得到执行Controller时的异常信息
	// 这里可记录操作日志
	public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {
		System.out.println("HandlerInterceptor1....afterCompletion");
	}

}
```
* 拦截器配置
在springmvc.xml中配置拦截器
```
<!-- 配置拦截器 -->
<mvc:interceptors>
	<mvc:interceptor>
		<!-- 所有的请求都进入拦截器 -->
		<mvc:mapping path="/**" />
		<!-- 配置具体的拦截器 -->
		<bean class="com.study.springmvc.interceptor.HandlerInterceptor1" />
	</mvc:interceptor>
	<mvc:interceptor>
		<!-- 所有的请求都进入拦截器 -->
		<mvc:mapping path="/**" />
		<!-- 配置具体的拦截器 -->
		<bean class="com.study.springmvc.interceptor.HandlerInterceptor2" />
	</mvc:interceptor>
</mvc:interceptors>
```
* 拦截流程测试
1 . 正常测试结果
```
HandlerInterceptor1....preHandle
HandlerInterceptor2....preHandle
HandlerInterceptor2....postHandle
HandlerInterceptor1....postHandle
HandlerInterceptor2....afterCompletion
HandlerInterceptor1....afterCompletion
```
2 . 中断流程测试
HandlerInterceptor1的preHandler方法返回false，HandlerInterceptor2返回true
```
HandlerInterceptor1....preHandle
```
HandlerInterceptor1的preHandler方法返回true，HandlerInterceptor2返回false
```
HandlerInterceptor1....preHandle
HandlerInterceptor2....preHandle
HandlerInterceptor1....afterCompletion
```
**总结**：
preHandle按拦截器定义顺序调用
postHandler按拦截器定义逆序调用
afterCompletion按拦截器定义逆序调用

postHandler在拦截器链内所有拦截器返成功调用
afterCompletion只有preHandle返回true才调用

* 拦截器应用
以用户登录为例：
```
	@RequestMapping(value = "/login.action",method = RequestMethod.GET)
	public String login(){
		return "login";
	}
	
	@RequestMapping(value = "/login.action",method = RequestMethod.POST)
	public String login(String username
			,HttpSession httpSession){
		httpSession.setAttribute("USER_SESSION", username);
		return "redirect:/item/itemlist.action";
	}
```
```
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object arg2) throws Exception {
		String requestURI = request.getRequestURI();
		if(!requestURI.contains("/login")){
			String username = (String) request.getSession().getAttribute("USER_SESSION");
			if(null == username){
				response.sendRedirect(request.getContextPath() + "/login.action");
				return false;
			}
		}
		return true;
	}
```
还需要在springmvc.xml配置拦截器

###可能遇到的问题
* 解决post乱码问题
在web.xml中加入:
```
	<!-- 解决post乱码问题 -->
	<filter>
		<filter-name>encoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<!-- 设置编码参是UTF8 -->
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encoding</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```
以上可以解决post请求乱码问题。

对于get请求中文参数出现乱码解决方法有两个：
修改tomcat配置文件添加编码与工程编码一致，如下：
```
<Connector URIEncoding="utf-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

另外一种方法对参数进行重新编码：
```
String userName  = new String(request.getParameter("userName").getBytes("ISO8859-1"),"utf-8");
```
ISO8859-1是tomcat默认编码，需要将tomcat编码后的内容按utf-8编码。
* 使用MappingJacksonValue解决jsonp请求跨域问题
```
		$.ajax({
			url : "http://localhost:8088/user/token/" + token,
			dataType : "jsonp",
			type : "GET",
			success : function(data){
				if(data.status == 200){
				}
			}
		});
```
```
	@RequestMapping(value="/user/token/{token}")
	@ResponseBody
	public Object getUserByToken(@PathVariable String token, String callback) {
		E3Result result = tokenService.getUserByToken(token);
		//响应结果之前，判断是否为jsonp请求
		if (StringUtils.isNotBlank(callback)) {
			//把结果封装成一个js语句响应
			MappingJacksonValue mappingJacksonValue = new MappingJacksonValue(result);
			mappingJacksonValue.setJsonpFunction(callback);
			return mappingJacksonValue;
		}
		return result;
	}
```
* 解决请求\*.html后缀无法返回json数据的问题
在springmvc中请求*.html不可以返回json数据。
修改web.xml，添加url拦截格式。
```
	<servlet-mapping>
		<servlet-name>cart-web</servlet-name>
		<url-pattern>*.action</url-pattern>
	</servlet-mapping>
```
###资料
[**Spring MVC官方4.2.4.RELEASE版本的文档**](http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/mvc.html)
[**Spring MVC 4.2.4.RELEASE 中文文档**](https://linesh.gitbooks.io/spring-mvc-documentation-linesh-translation/content/)
