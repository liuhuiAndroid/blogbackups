### 一、什么是FreeMarker
>FreeMarker 是一款模板引擎：即一种基于模板和要改变的数据，并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。
* 原理图
![image.png](https://upload-images.jianshu.io/upload_images/1956963-d103f03ed871b4dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 二、Freemarker的使用方法
* 添加Freemarker的jar包
```
        <dependency>
			<groupId>org.freemarker</groupId>
			<artifactId>freemarker</artifactId>
			<version>2.3.23</version>
		</dependency>
```
测试方法
```
	@Test
	public void testFreeMarker() throws Exception {
		//1、创建一个模板文件
		//2、创建一个Configuration对象
		Configuration configuration = new Configuration(Configuration.getVersion());
		//3、设置模板文件保存的目录
		configuration.setDirectoryForTemplateLoading(new File("E:/web workspace10/test-mq/test-web/src/main/webapp/WEB-INF/ftl"));
		//4、模板文件的编码格式，一般就是utf-8
		configuration.setDefaultEncoding("utf-8");
		//5、加载一个模板文件，创建一个模板对象。
		Template template = configuration.getTemplate("hello.ftl");
		//6、创建一个数据集。可以是pojo也可以是map。推荐使用map
		Map data = new HashMap<>();
		data.put("hello", "hello freemarker!");
		//7、创建一个Writer对象，指定输出文件的路径及文件名。
		Writer out = new FileWriter(new File("D:/temp/freemarker/hello.txt"));
		//8、生成静态页面
		template.process(data, out);
		//9、关闭流
		out.close();
	}
```
hello.ftl
```
${hello}
```
### 三、Freemarker模板的语法
* 访问map中的key
```
${key}
```
* 访问pojo中的属性
```
${student.id}
```
* 取集合中的数据和下标
```
<#list stuList as stu>
        ${stu_index}
</#list>
```
* 判断
```
		<#if stu_index % 2 == 0>
		<tr bgcolor="red">
		<#else>
		<tr bgcolor="green">
		</#if>
```
* 日期类型格式化
```
	<!-- 可以使用?date,?time,?datetime,?string(parten)-->
	当前日期：${date?string("yyyy/MM/dd HH:mm:ss")}<br>
```
* null值的处理
```
	<#if val??>
	val中有内容
	<#else>
	val的值为null
	</#if>
```
* include标签
```
	引用模板测试：<br>
	<#include "hello.ftl">
```

### 四、Freemarker整合Spring
* 引入Freemarker的jar包
```
		<dependency>
			<groupId>org.freemarker</groupId>
			<artifactId>freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jms</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
		</dependency>
```
* springmvc.xml中配置FreeMarker
```
	<!-- 配置freemarker -->
	<bean id="freemarkerConfig"
		class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
		<property name="templateLoaderPath" value="/WEB-INF/ftl/" />
		<property name="defaultEncoding" value="UTF-8" />
	</bean>
```
* 编写Controller
```
@Controller
public class HtmlGenController {
	
	@Autowired
	private FreeMarkerConfigurer freeMarkerConfigurer;

	@RequestMapping("/genhtml")
	@ResponseBody
	public String genHtml()throws Exception {
		// 1、从spring容器中获得FreeMarkerConfigurer对象。
		// 2、从FreeMarkerConfigurer对象中获得Configuration对象。
		Configuration configuration = freeMarkerConfigurer.getConfiguration();
		// 3、使用Configuration对象获得Template对象。
		Template template = configuration.getTemplate("hello.ftl");
		// 4、创建数据集
		Map dataModel = new HashMap<>();
		dataModel.put("hello", "1000");
		// 5、创建输出文件的Writer对象。
		Writer out = new FileWriter(new File("D:/temp/freemarker/spring-freemarker.html"));
		// 6、调用模板对象的process方法，生成文件。
		template.process(dataModel, out);
		// 7、关闭流。
		out.close();
		return "OK";
	}
	
}
```

### 五、案例：商品详情页面静态化

###资料
[FreeMarker官方文档](https://freemarker.apache.org/)
[FreeMarker中文文档](http://freemarker.foofun.cn/)
