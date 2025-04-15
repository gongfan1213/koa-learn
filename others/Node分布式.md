### 分布式系统
- **定义与目的**：一组通过网络连接的计算机系统，用于解决单台计算机无法承担的计算和存储任务，可处理海量数据（至上T级别 ）。当单个节点无法满足数据增长需求时，采用分布式系统，将任务拆分到多台服务器并行处理后汇总 。
- **挑战与应对**：依赖网络连接，存在网络不稳定问题，通过冗余保障系统正常运行；由多个节点构成，易出现节点故障，引发系统不可用；网络超时是难题 。
- **特性**：可扩展性，能随时扩展；透明性，用户无需关心内部实现；可用性和可靠性，保证系统大部分时间可用；数据冗余，确保多节点数据一致 。
- **基本架构**：PC和App访问Nginx，Nginx作为网关将信息发送到多台WebServer服务器，WebServer实现Session分布式。WebServer间通过HTTP通信，应用间借助MQ等消息中间件交互 。系统包含多台数据库（dbProxy为访问网关 ）、zookeeper注册中心、cache缓存服务、logging日志系统、Hadoop数据处理中心 。

### 负载均衡
- **定义与功能**：一种计算机技术，在多个计算机、计算机集群等资源间分配工作负载，使吞吐量最大化、响应时间最小化，防止过载，提高互联网服务的可用性和可靠性 。
- **分类与特点**：按层次分为2层、3层、4层、7层负载均衡；按软硬分为软负载均衡（成本低、灵活但易受服务器影响 ）和硬负载均衡（性能好但成本高 ） 。
- **Node.js实现**：通过cluster模块实现。主进程引入cluster、http、os模块，判断是否为主进程，若是则打印主进程信息，创建与CPU数目相同的子进程；子进程创建http服务器监听8000端口 。多进程监听同一端口实现负载均衡，进程间主要用IPC通信 。

### 无状态化
- **应用拆分原因**：将应用拆分为微服务应对高并发，单个进程难以承受大流量，拆分后各进程承担特定工作，共同承载并发 。
- **状态处理问题**：单体变分布式时，本地存储状态（内存或硬盘 ）会成瓶颈。用户信息若存于单一进程，会限制多进程流量分发 。
- **系统状态划分**：系统分有状态和无状态部分，业务逻辑多在无状态部分，有状态部分存于Redis、MQ、Hadoop等中间件 。
- **数据无状态化类型**：包括结构化数据无状态化、会话数据无状态化（如Session无状态化、Cookie无状态化，可用token实现前后端分离 ） 。以Redis存储Node.js中Session为例，安装Express框架及相关依赖，引入模块，配置Redis客户端和Session中间件，实现Session分布式存储 。

### 雪花算法
Twitter公司采用的算法，用于生成全局唯一且趋势增长的ID。由4部分组成：1bit始终为0无实际作用；41bit时间戳，精确到毫秒，可容纳约69年事件；10bit工作机器id，高位5bit是数据中心ID，低位5bit是工作节点ID，最多容纳1024个节点 ；12bit序列号，每个节点每毫秒从0累加，最多到4095，一毫秒内可生成1024 * 4096 = 4194304个ID 。

### 文件图片数据无状态化
可使用阿里云的OSS、腾讯云的COS等云计算厂商服务实现，相关介绍可访问对应官网，如阿里云OSS：https://www.aliyun.com/product/oss 。

### 非结构化数据无状态化
可使用MongoDB实现，具体详见MongoDB相关章节。

### 远程过程调用（RPC）
- **定义**：把本地调用逻辑处理过程放在远程机器上执行，涉及通信框架、通信协议、序列化和反序列化 。
- **调用步骤**：Client通过本地RPC代理调用接口；本地代理将服务名、方法名和参数等转换为RPC Request对象；RPC框架按协议将对象转为二进制，通过TCP传给Server；Server接收后反序列化对象，找到对应方法执行并封装结果为RPC Response对象；RPC框架将Response对象转为二进制传给Client；Client解码并通过代理返回结果 。
- **示例（gRPC框架 ）**：使用git下载官方示例代码（git clone -b v1.35.0 https://github.com/grpc/grpc ），进入相关目录（cd grpc/examples/node/dynamic_codegen ），安装依赖（npm install ），启动Server端（node.\greeter_server.js ）和client端（node.\greeter_client.js ） 。若client端输出“Greeting: Hello world”，表示示例程序执行基本完成。还可编辑examples/protos/helloworld.proto文件在Server服务里增加SayHelloAgain等RPC函数 。

### 1. RPC 函数扩展
在gRPC的Greeter服务中新增SayHelloAgain函数 。
- **服务端实现**：修改greeter_server.js文件 ，添加sayHelloAgain函数，接收HelloRequest类型的call参数和回调函数callback 。函数内从call.request中获取name ，构造包含问候语的HelloReply对象返回给客户端 。main函数中使用server.addService将新增函数注册到服务中，绑定地址并启动服务 。
- **客户端实现**：修改greeter_client.js文件 ，添加sayHelloAgain函数调用，传入参数{name: 'you'}和回调函数 。回调函数在接收到服务端响应后，打印响应中的message 。运行时先启动服务端（node greeter_server.js ），再启动客户端（node greeter_client.js ），客户端会输出“Greeting: Hello you”和“Greeting: Hello again, you”，表明远程调用成功 。

### 2. 消息中间件（以RabbitMQ为例 ）
消息中间件用于解决分布式系统间消息传递问题 ，如RabbitMQ、Kafka等 。以消息发送方send发送消息到队列Queue ，Queue再分发给接收方Receiver为例 。
- **安装RabbitMQ**：在CentOS系统下使用Docker安装 。先拉取镜像（docker pull rabbitmq:3.7.7 - management ），再运行容器（docker run -d --name rabbitmq3.7.7 -p 5672:5672 -p 15672:15672 -v 'pwd'/data:/var/lib/rabbitmq --hostname myRabbit -e RABBITMQ_DEFAULT_VHOST=my_vhost -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.7.7 - management ），映射端口、挂载目录、设置主机名、虚拟主机、用户名和密码 。访问15672端口 ，出现相应界面则安装完成 。
- **生产者代码（send.js ）**：引入amqplib模块 ，使用amqp.connect连接RabbitMQ服务器 ，连接成功后创建通道（connection.createChannel ）。在通道回调中声明队列（channel.assertQueue ），指定队列为'hello' ，非持久化 。然后使用channel.sendToQueue发送消息'Hello world' ，最后通过setTimeout在500毫秒后关闭连接并退出进程 。运行代码会在控制台输出“[x] Sent Hello World!” 。 
- **消费者代码（receive.js ）**：同样引入amqplib模块并连接服务器、创建通道 。声明队列'hello' ，使用channel.consume监听队列消息 ，传入队列名、回调函数和noAck参数（设置为true表示不发送确认信息到生产者 ）。回调函数中打印接收到的消息内容 。运行代码控制台会先输出“[*] Waiting for messages in hello. To exit press CTRL+C” ，接收到消息后输出“[x] Received Hello World!” 。



