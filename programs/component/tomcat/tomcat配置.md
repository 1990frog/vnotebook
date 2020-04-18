[TOC]

HTTP 的 keepalive 是双方通过 "Connection: keep-alive" 数据头来建立长连接，每次发送完请求不关闭套接字，而是继续下一次循环的 recv()，等待对端继续发送数据。如果设置了超时时间，那么 recv() 返回，关闭连接。

# threadPriority
线程优先级，默认是5（Thread.NORM_PRIORITY）
# deamon
（boolean）是否deamon线程，默认为true
# namePrefix
（String）线程前缀
# maxThreads
（int）线程池中的最大线程数
# minSpareThreads
（int）最小线程数（线程空间超过一段时间就会被回收）
# maxQueueSize
（int）线程池中任务队列的最大长度
# prestartminSpareThreads
（boolean）是在线程池启动时就创建minSpareThreads个线程



# Springboot
accept-count:等待队列长度，默认100
max-connections:最大可被连接数，默认10000
max-threads:最大工作线程数，默认200
min-spare-threads:最小工作线程数，默认10

默认配置下，连接操过10000后出现拒绝连接情况
默认配置下，触发的请求超过200+100后拒绝处理


+ keepAliveTimeOut：多少毫秒后不影响的断开keepalive
+ maxKeepAliveRequests:多少次请求后keepalive断开失效
+ 使用WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>定制化内嵌tomcat配置

spring-configuration-metadata.json文件
查看各个节点的配置

对一个4核8g内存的虚拟机经验值是800 max threads

连接有信息要处理时才会用到线程 一般线程数会比连接数小很多

一般计算，1核心1gb内存算100个

线程数对系统不是越多越好，是一个由上升到下降的状态，线程越多对cpu消耗越大，反而发挥不出效果，扩大线程池是为了达到最佳状态，限制线程数是为了寻找最好的效果点

@SpringBootConfiguration
public class WebServerConfiguration {

    @Bean
    public ConfigurableServletWebServerFactory webServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setPort(11359);//端口号
        factory.setUriEncoding(Charset.forName("utf-8"));//编码
        factory.addConnectorCustomizers(new MyTomcatConnectorCustomizer());
        return factory;
    }
    
    class MyTomcatConnectorCustomizer implements TomcatConnectorCustomizer {

        @Override
        public void customize(Connector connector) {
            // TODO Auto-generated method stub
            Http11NioProtocol handler = (Http11NioProtocol)connector.getProtocolHandler();
            
            handler.setAcceptCount(2000);//排队数
                        
            handler.setMaxConnections(5000);//最大连接数
            
            handler.setMaxThreads(2000);//线程池的最大线程数
            
            handler.setMinSpareThreads(100);//最小线程数
                        
            handler.setConnectionTimeout(30000);//超时时间                    
        }
    }
}
