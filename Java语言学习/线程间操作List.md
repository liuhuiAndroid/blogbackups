- [例子来自 Java多线程操作List](http://wuwenjun0919-msn-com.iteye.com/blog/2174652)
- 接下来是超详细分析

```
/**
 * 多线程处理List 
 * @author lh
 *
 */
public class MultiThread {
	
	public static final List<Long> list  = Collections.synchronizedList(new ArrayList<Long>());
	
	public static void main(String[] args) {
		for(int i = 1;i<=100;i++){
			list.add(Long.valueOf(i));
		}
		
		MyThread myThread = new MultiThread().new MyThread(); 
		Thread t1 = new Thread(myThread); 
		t1.setName("线程1"); 
		t1.start(); 

		Thread t2 = new Thread(myThread); 
		t2.setName("线程2"); 
		t2.start(); 

		Thread t3 = new Thread(myThread); 
		t3.setName("线程3"); 
		t3.start(); 

		Thread t4 = new Thread(myThread); 
		t4.setName("线程4"); 
		t4.start(); 
	}
	
	
	public class MyThread implements Runnable{

		@Override
		public void run() {
			for(int i = 0;i<list.size();i++){
				// 同步list，打印数据并删除该数据 
				synchronized (list) {
					try {
						//当前线程睡眠，让其它线程获得执行机会 
						Thread.sleep(100);
					} catch (Exception e) {
						// TODO: handle exception
					}
					System.out.print(Thread.currentThread().getName() + ":" + list.get(i)+"\n");
					list.remove(i);
				}
			}
		}
		
	}
}
```
> - 好奇为什么在有synchroniezed方法的同时会出现Collections.synchronizedList
> - Collections.synchronizedList可以得到本身不是线程安全的容易的线程安全的状态
> - 线程安全仅仅指的是如果直接使用它提供的函数，比如:add(obj);或者poll(obj);，这样我们自己不需要做任何同步
> - Collections.synchronizedList返回的线程安全的List内部使用的锁是list
> - 可以实现线程安全性
- 分析参考[Collections.synchronizedList](https://my.oschina.net/infiniteSpace/blog/305425)
- 参考[Java的容器的线程安全 ](http://blog.sina.com.cn/s/blog_5efa3473010129pw.html)


#有空看一本java并发编程的书再接着更新哈
