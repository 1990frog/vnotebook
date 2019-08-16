# study_note

组件自动装配
激活：@EnableAutoConfiguration
配置：/META-INF/spring.factories
实现：XXXAutoConfiguration

SpirngBoot
组件自动装配：规约大于配置，专注核心业务
外部化配置：一次构建、按需调配、到处运行
嵌入式容器：内置容器、无需部署、独立运行
Spring Boot Starter:简化依赖、按需装配、自我包含
Production-Ready：一站式运维、生态无缝整合

SpringBoot难精
组件自动装配：模式注解、@Enable模块、条件装配、加载机制
外部化配置：Environment抽象、生命周期、破坏性变更
嵌入式容器：Servlet Web容器、Reactive Web容器
Spring Boot Starter：依赖管理、装配条件、装配顺序
Production-Ready：健康检查、数据指标、@Endpoint管控

SpringBoot与JavaEE规范
Web：Servlet（JSR-315、JSR340）
SQL：JDBC（JSR-221）
数据校验：Bean Validation（JSR303、JSR349）
缓存：Java Caching API（JSR-107）
WebSockets：Java API for WebSocket（JSR-356）
Web Services：JAX-WS（JSR-224）
Java管理：JMS（JSR3）
消息：JMS（JSR-914）