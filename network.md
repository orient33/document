## okhttp-3.14.x

 ### 1. dispatch

```java
//异步--添加到dispatch的等待队列,再到线程池
  client.dispatcher().enqueue(new AsyncCall(responseCallback));//->
  readyAsyncCalls.add(call);//->
  asyncCall.executeOn(executorService());//最后扔到dispatcher线程池
//同步--添加到running队列
  client.dispatcher().executed(this);
  runningSyncCalls.add(call);
```

### 2.chain 责任链--拦截器

    最后都会确保子线程执行.  RealCall.execute() -> getResponseWithInterceptorChain() //即得到response
    
    依次添加 自定义interceptors, RetryAndFollowUpInterceptor().

```java
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(new RetryAndFollowUpInterceptor(client));
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));//打开url链接
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));//向服务端call
```
### 3. CallServerInterceptor, ConnectInterceptor

  Exchange 传输一次http请求 得到response pair, 写header, requestBody 到 Sink(即向服务端写数据),,发完数据,  finishRequest(),再 拿 exchange.responseHeaders
  //见 CallServerInterceptor.intercept()方法.
  ExchangeCodec .管理链接 //分Http1ExchangeCodec ,Http2ExchangeCodec

### 4. 服务端sink/流 来自 socket, 

在RealConnect() 得到的sink
  ExchangeFinder.findConnection()

 RealConnectionPool..线程池. (0-MAX-60秒)
 最终使用 java.net.Socket.connect(InetSocketAddress, timeout);//链接socket
 soket 封装到 Okio的 source 与sink. ( source与sink 分别是 InputStream, OutputStream的封装)
即最后依赖的为Socket的 inputStream,outputStream
sink-接收器

### 5.socket来自于?

okhttp3.Dns. --SYSTEM -> java.net.InetAddress.getAllByName(hostname)
 //即根据hostname得到 ip地址,
 SocketFactory.DefaultSocketFactory....new Socket(InetAddress, port)
  Transmitter..

### 6.总结

 a. 根据host解析出ip.(使用java.net.InetAddress.getAllByName(host))
 b. 创建Socket()  new Socket()
 c. 根据socket创建source/sink, 输入/输出流.
 d. 向sock写请求数据, 再读取响应数据.