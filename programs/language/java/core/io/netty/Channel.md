[TOC]

# 根接口ChannelHandler
```java
public interface ChannelHandler {

    // 当把ChannelHandler添加到ChannelPipeline中时被调用
    void handlerAdded(ChannelHandlerContext ctx) throws Exception;
    // 当从ChannelPipeline中移除ChannelHandler时被调用
    void handlerRemoved(ChannelHandlerContext ctx) throws Exception;

    @Deprecated
    void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception;
}
```
# 分化出两种行为”进“和”出“
+ ChannelInboundHandler：处理入站数据及各种状态变化
+ ChannelOutboundHandler：处理出站数据并且允许拦截所有的操作

SimpleChannelInboundHandler
SimpleCHannelOutboundHandler