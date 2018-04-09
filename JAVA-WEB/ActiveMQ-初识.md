###一、什么是[ActiveMQ](http://activemq.apache.org/)
>[Apache ActiveMQ](http://activemq.apache.org/) ™ is the most popular and powerful open source messaging and [Integration Patterns](http://activemq.apache.org/enterprise-integration-patterns.html) server.
Apache ActiveMQ™是最流行的，能力强劲的开源消息总线

###二、[ActiveMQ](http://activemq.apache.org/)的消息形式
消息的传递有两种类型：
一种是**点对点**的，即一个生产者和一个消费者一一对应；
另一种是**发布/订阅模式**，即一个生产者产生消息并进行发送后，可以由多个消费者进行接收。
JMS定义了五种不同的消息正文格式，以及调用的消息类型，允许你发送并接收以一些不同形式的数据，提供现有消息格式的一些级别的兼容性。
* StreamMessage -- Java原始值的数据流
* MapMessage  一套名称-值对
* **TextMessage**  一个字符串对象
* ObjectMessage  一个序列化的 Java对象
* BytesMessage  一个字节的数据流

###三、[ActiveMQ](http://activemq.apache.org/)的安装
```
1.上传apache-activemq-5.12.0-bin.tar.gz到服务器
tar -zxvf apache-activemq-5.12.0-bin.tar.gz -C /usr/local
2.启动 bin/activemq start
3.查看状态 bin/activemq status
4.关闭 bin/activemq stop
5.进入管理后台
http://192.168.10.179:8161/admin
用户名：admin
密码：admin
```
注意：如果ActiveMQ整合spring使用建议使用5.11.2

###四、ActiveMQ的使用方法
* 添加依赖
```
		 <dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-all</artifactId>
		</dependency>
```
* Queue
1. Producer
```
	@Test
	public void testQueueProducer() throws Exception {
		//1、创建一个连接工厂对象，需要指定服务的ip及端口。
		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.10.179:61616");
		//2、使用工厂对象创建一个Connection对象。
		Connection connection = connectionFactory.createConnection();
		//3、开启连接，调用Connection对象的start方法。
		connection.start();
		//4、创建一个Session对象。
		//第一个参数：是否开启事务。如果true开启事务，第二个参数无意义。一般不开启事务false。
		//第二个参数：应答模式。自动应答或者手动应答。一般自动应答。
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		//5、使用Session对象创建一个Destination对象。两种形式queue、topic，现在应该使用queue
		Queue queue = session.createQueue("test-queue");
		//6、使用Session对象创建一个Producer对象。
		MessageProducer producer = session.createProducer(queue);
		//7、创建一个Message对象，可以使用TextMessage。
		/*TextMessage textMessage = new ActiveMQTextMessage();
		textMessage.setText("hello Activemq");*/
		TextMessage textMessage = session.createTextMessage("hello activemq");
		//8、发送消息
		producer.send(textMessage);
		//9、关闭资源
		producer.close();
		session.close();
		connection.close();
	}
```
2.Consumer
```
	@Test
	public void testQueueConsumer() throws Exception {
		//创建一个ConnectionFactory对象连接MQ服务器
		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.10.179:61616");
		//创建一个连接对象
		Connection connection = connectionFactory.createConnection();
		//开启连接
		connection.start();
		//使用Connection对象创建一个Session对象
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		//创建一个Destination对象。queue对象
		Queue queue = session.createQueue("test-queue");
		//使用Session对象创建一个消费者对象。
		MessageConsumer consumer = session.createConsumer(queue);
		//接收消息
		consumer.setMessageListener(new MessageListener() {
			@Override
			public void onMessage(Message message) {
				//打印结果
				TextMessage textMessage = (TextMessage) message;
				String text;
				try {
					text = textMessage.getText();
					System.out.println(text);
				} catch (JMSException e) {
					e.printStackTrace();
				}
			}
		});
		//等待接收消息
		System.in.read();
		//关闭资源
		consumer.close();
		session.close();
		connection.close();
	}
```
* Topic
1. Producer
```
	@Test
	public void testTopicProducer() throws Exception {
		//1、创建一个连接工厂对象，需要指定服务的ip及端口。
		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.10.179:61616");
		//2、使用工厂对象创建一个Connection对象。
		Connection connection = connectionFactory.createConnection();
		//3、开启连接，调用Connection对象的start方法。
		connection.start();
		//4、创建一个Session对象。
		//第一个参数：是否开启事务。如果true开启事务，第二个参数无意义。一般不开启事务false。
		//第二个参数：应答模式。自动应答或者手动应答。一般自动应答。
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		//5、使用Session对象创建一个Destination对象。两种形式queue、topic，现在应该使用topic
		Topic topic = session.createTopic("test-topic");
		//6、使用Session对象创建一个Producer对象。
		MessageProducer producer = session.createProducer(topic);
		//7、创建一个Message对象，可以使用TextMessage。
		/*TextMessage textMessage = new ActiveMQTextMessage();
		textMessage.setText("hello Activemq");*/
		TextMessage textMessage = session.createTextMessage("topic message");
		//8、发送消息
		producer.send(textMessage);
		//9、关闭资源
		producer.close();
		session.close();
		connection.close();
	}
```
2.Consumer
```
	@Test
	public void testTopicConsumer() throws Exception {
		//创建一个ConnectionFactory对象连接MQ服务器
		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://192.168.10.179:61616");
		//创建一个连接对象
		Connection connection = connectionFactory.createConnection();
		//开启连接
		connection.start();
		//使用Connection对象创建一个Session对象
		Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
		//创建一个Destination对象。topic对象
		Topic topic = session.createTopic("test-topic");
		//使用Session对象创建一个消费者对象。
		MessageConsumer consumer = session.createConsumer(topic);
		//接收消息
		consumer.setMessageListener(new MessageListener() {
			@Override
			public void onMessage(Message message) {
				//打印结果
				TextMessage textMessage = (TextMessage) message;
				String text;
				try {
					text = textMessage.getText();
					System.out.println(text);
				} catch (JMSException e) {
					e.printStackTrace();
				}
			}
		});
		System.out.println("topic消费者1启动。。。。");
		//等待接收消息
		System.in.read();
		//关闭资源
		consumer.close();
		session.close();
		connection.close();
	}
```

###四、ActiveMQ整合Spring
* 导入jar
```
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jms</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
		</dependency>
```
* 配置ConnectionFactory
新建applicationContext-activemq.xml
```
	<!-- 真正可以产生Connection的ConnectionFactory，由对应的 JMS服务厂商提供 -->
	<bean id="targetConnectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
		<property name="brokerURL" value="tcp://192.168.25.161:61616" />
	</bean>
	<!-- Spring用于管理真正的ConnectionFactory的ConnectionFactory -->
	<bean id="connectionFactory"
		class="org.springframework.jms.connection.SingleConnectionFactory">
		<!-- 目标ConnectionFactory对应真实的可以产生JMS Connection的ConnectionFactory -->
		<property name="targetConnectionFactory" ref="targetConnectionFactory" />
	</bean>
```
* 配置生产者。
* 在Spring容器中配置Destination
```
	<!-- 配置生产者 -->
	<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->
	<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
		<!-- 这个connectionFactory对应的是我们定义的Spring提供的那个ConnectionFactory对象 -->
		<property name="connectionFactory" ref="connectionFactory" />
	</bean>
	<!--这个是队列目的地，点对点的 -->
	<bean id="queueDestination" class="org.apache.activemq.command.ActiveMQQueue">
		<constructor-arg>
			<value>spring-queue</value>
		</constructor-arg>
	</bean>
	<!--这个是主题目的地，一对多的 -->
	<bean id="topicDestination" class="org.apache.activemq.command.ActiveMQTopic">
		<constructor-arg value="itemAddTopic" />
	</bean>
```
* 测试发送消息
```
	@Test
	public void sendMessage() throws Exception {
		//初始化spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-activemq.xml");
		//从容器中获得JmsTemplate对象。
		JmsTemplate jmsTemplate = applicationContext.getBean(JmsTemplate.class);
		//从容器中获得一个Destination对象。
		Destination destination = (Destination) applicationContext.getBean("queueDestination");
		//发送消息
		jmsTemplate.send(destination, new MessageCreator() {
			@Override
			public Message createMessage(Session session) throws JMSException {
				return session.createTextMessage("send activemq message");
			}
		});
	}
```
可以注入的方式
```
	@Autowired
	private JmsTemplate jmsTemplate;
	@Resource
	private Destination topicDestination;
```
* 测试接收消息
```
public class MyMessageListener implements MessageListener {
	@Override
	public void onMessage(Message message) {
		try {
			TextMessage textMessage = (TextMessage) message;
			//取消息内容
			String text = textMessage.getText();
			System.out.println(text);
		} catch (JMSException e) {
			e.printStackTrace();
		}
	}
}
```
```
	<!-- 接收消息 -->
	<!-- 配置监听器 -->
	<bean id="myMessageListener" class="com.java.lh.MyMessageListener" />
	<!-- 消息监听容器 -->
	<bean class="org.springframework.jms.listener.DefaultMessageListenerContainer">
		<property name="connectionFactory" ref="connectionFactory" />
		<property name="destination" ref="queueDestination" />
		<property name="messageListener" ref="myMessageListener" />
	</bean>
```
```
	@Test
	public void testQueueConsumerSpring() throws Exception {
		//初始化spring容器
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext-activemq.xml");
		//等待
		System.in.read();
	}
```
