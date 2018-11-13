---
title: 从socket api的几个方法看tcp机制
date: 2018-10-29 20:37:21
tags: 
   - tcp
   - netty
---
## 背景问题
在使用rpc框架时，设置socket参数时会使用到下面的api：

soccket.setReceiveBufferSize()------设置接受数据的缓冲区大小

socket.setSendBufferSize()----------设置发送数据的缓冲区大小
<!-- more -->

socket.setTcpNoDelay()--------------是否开启Nagle算法，true开启，false关闭

socket.setSoLinger()----------------close socket时 发送队列如何处理数据，丢弃或者继续发送，通常需要关闭，保持默认行为，由系统接管保证发送数据完成

socket.setSoTimeout()---------------对于服务端来说，accept空等客户端连接超过则异常，对于客户端来说，read读数据流取数据最长等待时间，超过则异常

socket.setKeepAlive()---------------默认的心跳保活机制，但在连接双方ESTABLISEHED之后两个小时左右上层没有数据传输的情况下，才会起作用。一般应用不使用此机制，需要自己做保活机制

socket.setTrafficClass()------------IP_TOS,设置服务类型，共四种：IPTOS_LOWCOST，IPTOS_RELIABILITY，IPTOS_THROUGHPUT，IPTOS_LOWDELAY，可组合使用

socket.setReuseAddress()------------是否重用处于TIME_WAIT状态的地址

socket.setPerformancePreferences(connectionTime,latency,bandwidth)---设置低连接时间/低延迟/高带宽相对的优先级

ServerSocket backlog----------------服务端请求处理线程满时，临时存放已完成三次握手的请求队列的长度，默认50

这些参数对我们在进行网络编程时起到了什么作用，不同的参数设置会有什么影响程序的表现呢？


## tcp协议

### 拥塞控制

针对网络全局行的控制，主要为了防止网络链路出现过载，导致出现网络吞吐量由于负荷增大而下降的情况。

#### 慢开始与拥塞避免

!["慢开始"](/img/tcp_slow_start.jpeg)


#### 快重传&快恢复

!["快重传"](/img/tcp_fast_retransmission.jpeg)

!["快恢复"](/img/tcp_fast_recovery.jpeg)

### 滑动窗口&流量控制

针对点对点通信质量的控制，利用滑动窗口实现在TCP连接上对发送方发送速率的控制，控制流量，是得自己能够及时接收数据

!["滑动窗口"](/img/tcp_flow_control.jpeg)


## 应用

### rpc框架中的使用

目前大部分的rpc framework大多采用netty中支持的tcp协议套件来进行包装开发，避免传统nio+socket这一层级的复杂操作，但最终还是作用于ServerSocket和Socket上。但具体各自框架为什么选择或者不选择设置某些选项，是需要仔细考究的点（后续补充）。

#### dubbo

server端(NettyServer)ServerSocketChannelConfig根据目标操作系统会有不同的实现

```java

        bootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childOption(ChannelOption.TCP_NODELAY, Boolean.TRUE) // 开启TCP_NODELAY
                .childOption(ChannelOption.SO_REUSEADDR, Boolean.TRUE) // 开启 SO_REUSEADDR
                .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
                        ch.pipeline()//.addLast("logging",new LoggingHandler(LogLevel.INFO))//for debug
                                .addLast("decoder", adapter.getDecoder())
                                .addLast("encoder", adapter.getEncoder())
                                .addLast("handler", nettyServerHandler);
                    }
                });
```

client端(NettyClient) SocketChannelConfig 根据操作系统的实现会有不同

```java
bootstrap.group(nioEventLoopGroup)
                .option(ChannelOption.SO_KEEPALIVE, true)  //开启keepalive
                .option(ChannelOption.TCP_NODELAY, true)   //开启TCP_NODELAY
                .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
                //.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, getTimeout())
                .channel(NioSocketChannel.class);

        if (getConnectTimeout() < 3000) {.  // 设置
            bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 3000);
        } else {
            bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, getConnectTimeout());
        }

```

#### sofa-rpc

server端（RpcServer）

```java
  this.bootstrap.group(bossGroup, workerGroup).channel(NioServerSocketChannel.class)
            .option(ChannelOption.SO_BACKLOG, SystemProperties.tcp_so_backlog())
            .option(ChannelOption.SO_REUSEADDR, SystemProperties.tcp_so_reuseaddr())
            .childOption(ChannelOption.TCP_NODELAY, SystemProperties.tcp_nodelay())
            .childOption(ChannelOption.SO_KEEPALIVE, SystemProperties.tcp_so_keepalive());

```
client端（RpcConnectionFactory）

```java
bootstrap.group(workerGroup).channel(NioSocketChannel.class)
            .option(ChannelOption.TCP_NODELAY, SystemProperties.tcp_nodelay())
            .option(ChannelOption.SO_REUSEADDR, SystemProperties.tcp_so_reuseaddr())
            .option(ChannelOption.SO_KEEPALIVE, SystemProperties.tcp_so_keepalive());
```




