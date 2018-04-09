今天打算从Java动态代理的例子入手，然后探究Java动态代理的实现原理。
首先来看一下代理模式的定义：
>代理模式（Proxy Pattern）：给某一个对象提供一个代理，并由代理对象控制对原对象的引用。代理模式是一种对象结构性模式。

###Java动态代理例子
>静态代理：每一个代理类编译之后都会生成一个class文件，代理类所实现的接口和所代理的方法都被固定
动态代理：能够让系统在运行时动态创建代理类

```
public interface AbstractUserDAO {
	public boolean findUserById(String userId); 
}
```
```
public interface AbstractDocumentDAO {
	public boolean deleteDocumentById(String documentId); 
}
```


```
public class UserDAO implements AbstractUserDAO{

	@Override
	public boolean findUserById(String userId) {
		if(userId.equals("张伟")){
			System.out.println("查询ID为"+userId+"的用户信息成功！");
			return true;
		}else{
			System.out.println("查询ID为"+userId+"的用户信息失败！");
			return false;
		}
	}

}
```

```
public class DocumentDAO implements AbstractDocumentDAO{

	@Override
	public boolean deleteDocumentById(String documentId) {
		if(documentId.equals("设计模式的艺术")){
			System.out.println("查询ID为"+documentId+"的文档信息成功！");
			return true;
		}else{
			System.out.println("查询ID为"+documentId+"的文档信息失败！");
			return false;
		}
	}

}
```
```
public class DAOLogHandler implements InvocationHandler{

	private Object object;
	
	public DAOLogHandler(Object object) {
		super();
		this.object = object;
	}

	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		beforeInvoke();
		Object invoke = method.invoke(object, args);
		afterInvoke();
		return invoke;
	}
	
	private void beforeInvoke() {
		Calendar calendar = new GregorianCalendar();
		int hour = calendar.get(Calendar.HOUR_OF_DAY);
		int minute = calendar.get(Calendar.MINUTE);
		int second = calendar.get(Calendar.SECOND);
		String time = hour + ":" + minute + ":" +second;
		System.out.println("方法调用开始！time = "+time);
	}
	
	private void afterInvoke() {
		System.out.println("方法调用结束！");
	}

}
```
```
public class Client {
	public static void main(String[] args) {
		InvocationHandler handler = null;
		
		AbstractUserDAO userDAO = new UserDAO();

		handler = new DAOLogHandler(userDAO);
		AbstractUserDAO proxy = null;
		
		//Proxy.newProxyInstance用于返回一个动态创建的代理类的实例
		//参数一：代理类的类加载器    参数二:代理类所实现接口的方法    参数三：所指派的调用处理程序类
		proxy = (AbstractUserDAO)Proxy.newProxyInstance(AbstractUserDAO.class.getClassLoader(),
				new Class[]{AbstractUserDAO.class},handler);
		proxy.findUserById("张伟");
		
		System.out.println("--------------------------------");
		System.out.println("---------------同理--------------");
		
		AbstractDocumentDAO documentDAO = new DocumentDAO();
		handler = new DAOLogHandler(documentDAO);
		AbstractDocumentDAO proxy_new = null;
		proxy_new = (AbstractDocumentDAO)Proxy.newProxyInstance(AbstractDocumentDAO.class.getClassLoader(),
				new Class[]{AbstractDocumentDAO.class},handler);
		proxy_new.deleteDocumentById("设计模式的艺术");
	}
}
```
编译并运行程序，输出结果：
```
方法调用开始！time = 13:46:34
查询ID为张伟的用户信息成功！
方法调用结束！
--------------------------------
---------------同理--------------
方法调用开始！time = 13:46:34
查询ID为设计模式的艺术的文档信息成功！
方法调用结束！
```

到这里，大家应该很好奇Proxy类和InvocationHandler类怎么实现的，这里实现了一个简单版的。
```
public interface InvocationHandler {
	public void invoke(Object o, Method m);
}
```
```
public class Proxy {
	
	public static Object newProxyInstance(Class infce, InvocationHandler h) throws Exception { //JDK6 Complier API, CGLib, ASM
			
		String methodStr = "";
		String rt = "\r\n";
		
		Method[] methods = infce.getMethods();
		
		for(Method m : methods) {
			methodStr += "@Override" + rt + 
						 "public void " + m.getName() + "() {" + rt +
						 "    try {" + rt +
						 "    Method md = " + infce.getName() + ".class.getMethod(\"" + m.getName() + "\");" + rt +
						 "    h.invoke(this, md);" + rt +
						 "    }catch(Exception e) {e.printStackTrace();}" + rt +
						
						 "}";
		}
		
		String src = 
			"package com.designpattern.proxyprinciple;" +  rt +
			"import java.lang.reflect.Method;" + rt +
			"public class $Proxy1 implements " + infce.getName() + "{" + rt +
			"    public $Proxy1(InvocationHandler h) {" + rt +
			"        this.h = h;" + rt +
			"    }" + rt +
			
			
			"    com.designpattern.proxyprinciple.InvocationHandler h;" + rt +
							
			methodStr +
			"}";
		String fileName = 
			"d:/src/com/designpattern/proxyprinciple/$Proxy1.java";
		File f = new File(fileName);
		FileWriter fw = new FileWriter(f);
		fw.write(src);
		fw.flush();
		fw.close();
		
		//compile
		JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
		StandardJavaFileManager fileMgr = compiler.getStandardFileManager(null, null, null);
		Iterable units = fileMgr.getJavaFileObjects(fileName);
		CompilationTask t = compiler.getTask(null, fileMgr, null, null, null, units);
		t.call();
		fileMgr.close();
		
		//load into memory and create an instance
		URL[] urls = new URL[] {new URL("file:/" + "d:/src/")};
		URLClassLoader ul = new URLClassLoader(urls);
		Class c = ul.loadClass("com.designpattern.proxyprinciple.$Proxy1");
		System.out.println(c);
		
		Constructor ctr = c.getConstructor(InvocationHandler.class);
		Object m = ctr.newInstance(h);
		
		return m;
	}
	
}
```

了解到retrofit源码也是使用了动态代理，有空研究下源码。
##源码
[源代码](https://github.com/liuhuiAndroid/design-pattern)

##参考
书籍[设计模式的艺术](https://book.douban.com/subject/20493657)
