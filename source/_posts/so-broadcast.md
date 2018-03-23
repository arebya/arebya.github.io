---
title: so_broadcast
date: 2018-03-23 10:53:38
tags: Netty,网络
---

Netty在开发基于udp协议的代码中server段设置的参数broadcast有什么意义？


```JAVA
public static void initServer() {
        EventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .channel(NioDatagramChannel.class)
                    .option(ChannelOption.SO_BROADCAST, true)
                    .handler(new UdpServerHandler());
            Channel channel = bootstrap.bind(AppConstants.SEARCH_PORT).sync().channel();
            channel.closeFuture().await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            group.shutdownGracefully();
        }
    }
 ```


 设置SO_BROADCAST选项为0，只接受单播数据报
 设置SO_BROADCASE选项为1，接受UDP广播