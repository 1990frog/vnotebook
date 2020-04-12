[TOC]

CommonFilter
+ UrlBlockHander：提供sentinel异常处理
+ RequestOriginHander：来源支持
+ UrlCleaner：重新定义资源名称

如果想全部自定义
```
sentinel.filter.enabled:false
```
关闭过滤器

可以用切面等等方法实现拓展sentinel