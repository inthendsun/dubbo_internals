![服务端 ChannelHandler 类图](images/Dubbo_As_Provider_Basic_Flow.png)



- NettyServer 接收到客户端请求 , 经过 Netty 框架的三个 Pipeline

  - decoder  --> `NettyCodecAdapter.InternalDecoder`    
  - encoder  --> `NettyCodecAdapter.InternalEncoder`    
  - nettyHandler (重点)： NettyHandler 对于请求委托给 MultiMessageHandler

  注： `url 中的 codec 是在 DubboProtocol 的createServer中 设置的 codec = dubbo，对应的是DubboCountCodec`

  

- NettyHandler

  - 方法channelConnected： 将 channel 添加到 Map 里。 `注：心跳要使用`
  - 方法channelDisconnected： 将 channel 移出 Map。

- MultiMessageHandler  > HeartBeatHandler > AllChannelHandler > DecodeHandler > HeadExchangeHandler > ExchangeHandleAdapter

  

  - MultiMessageHandler 

  - HeartBeatHandler 

    - 设置每一个 Channel  最近接收和发送的时间

    - 接收到的包如果是心跳请求包，根据请求类型是否需要响应，决定是否响应。

    - 接收到的心跳包如果是我方发起的心跳请求对端的响应，

    - 接收是心跳请求或心跳响应就此结束。

      `twoway 的是意思是否是需要响应，客户端请求的时候如果设置false，那么服务端也不用费劲巴拉的响应了`

  - AllChannelHandler   使用线程池处理请求

    - connected 转发
    - disconnected 转发
    - received 转发
    - caught 转发

  - DecodeHandler 

  - HeadExchangeHandler

  - DubboProtocol 的内部类 `ExchangeHandlerAdapter`

    - received(Channel channel, Object message)

      > 根据 channel 构建查找的 key ，格式：com.alibaba.dubbo.demo.DemoService:20880， 从 DubboProtocol 的exporterMap 根据 key 查找到 DubboExporter（可以获取到 invoker,进而调用具体的接口实现类）

  

