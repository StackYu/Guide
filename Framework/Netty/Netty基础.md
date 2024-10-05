# 1. HelloWorld

```java
// 服务端
public class HelloNettyServer {
    public static void main(String[] args) {
        // 服务启动类
        new ServerBootstrap()
                // 选择线程池和selector
                .group(new NioEventLoopGroup())
                // 选择Server实现类型
                .channel(NioServerSocketChannel.class)
                // 添加处理器，ChannelInitializer为检测连接，成功只执行一次，添加后续处理器
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel nioSocketChannel) throws Exception {
                        // 添加解码器
                        nioSocketChannel.pipeline().addLast(new StringDecoder());
                        // 处理上一次处理器结果
                        nioSocketChannel.pipeline().addLast(new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                System.out.println(msg);
                            }
                        });
                    }
                })
                // 绑定端口
                .bind(8080);
    }
}

// 客户端
public class HelloNettyClient {
    public static void main(String[] args) throws InterruptedException {
        // 客户端启动类
        new Bootstrap()
                // 初始化线程和Selector
                .group(new NioEventLoopGroup())
                // Socket类型
                .channel(NioSocketChannel.class)
                // 处理器
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel channel) throws Exception {
                        channel.pipeline().addLast(new StringEncoder());
                    }
                })
                // 连接对象
                .connect("127.0.0.1", 8080)
                .sync()
                .channel()
                // 发送内容
                .writeAndFlush("Hello Netty! 你好！");
    }
}
```