# RPC

  一个可插拔式高可用 RPC 框架，分布式服务框架。
  
  RPC is a highly available pluggable architecture for remote calls.

### RPC 中文简介
1. RPC 基于 Java 编写，网络通信依赖与 netty，http，socket。
2. 支持基于配置的底层协议切换，可以选择 netty，http，socket。
3. 基于 Spring 开发，接口代理类自动注入到客户端，使用 @Autowired 注入即可，用户无需关注底层实现。
4. 支持 Spring xml 格式配置，通过 xml 完成代理类注入，服务端启动，通信协议选择。
5. 服务端使用线程池提高并发能力。
6. 客户端使用 channel 缓存提高并发能力。
7. 支持多序列化协议，多负载均衡协议选择。

开发中：
1. 加入 redis 注册中心，以及对应的服务治理容错机制。
2. 监控模块。
3. Spring 启动逻辑优化。

### RPC introduction
1. RPC is written in Java, and network communication depends on netty, http, socket.
2. Support configuration-based underlying protocol switching, you can choose netty, http, socket.
3. Based on Spring development, the interface proxy class is automatically injected into the client, using @Autowired injection, users do not need to pay attention to the underlying implementation.
4. Support Spring xml format configuration, complete proxy class injection through xml, server startup, communication protocol selection.
5. Server-side thread pool improves concurrency.
6. The client uses the channel cache to improve concurrency.
7. Support multi-serialization protocol, multi-load balancing protocol selection.

In development:
1. Add the redis registry and the corresponding service governance fault tolerance mechanism.
2. Monitoring module.
3. Spring starts logic optimization.

### 结构图
![avatar](https://github.com/PaulWang92115/RPC/blob/PAUL_RELEASE_1906/doc/20190630164543928.png)

### 性能表现
首先说明，性能表现测试根据不同的机器和不同的网络环境可能会有所不同，下面的测试结果是基于我自己的机器的。
我的电脑最多起 2000 个并发线程，多了就 OOM 了，在公司的电脑尝试过起 10000 个并发线程，没有任何问题，下面看 2000 个并发线程的表现。
测试类：
```java
	public static void main(String[] args) throws Exception {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("rpc.xml");
		//并行度10000
		int parallel = 2000;

		//开始计时
		long a1 = System.currentTimeMillis();

		CountDownLatch signal = new CountDownLatch(1);
		CountDownLatch finish = new CountDownLatch(parallel);

		for (int index = 0; index < parallel; index++) {
			CalcParallelRequestThread client = new CalcParallelRequestThread(signal, finish, index,applicationContext);
			new Thread(client).start();
		}

		//n个并发线程瞬间发起请求操作
		signal.countDown();
		finish.await();

		long a2 = System.currentTimeMillis();

		String tip = String.format("RPC调用总共耗时: [%s] 毫秒", a2 - a1);
		System.out.println(tip);

	}
```
![avatar](https://github.com/PaulWang92115/RPC/blob/PAUL_RELEASE_1906/doc/countdown.png)
2000 并发 1秒多，还是比较快的。
