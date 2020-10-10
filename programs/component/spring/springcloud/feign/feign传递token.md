[TOC]

@RequestHeader

![](https://gitee.com/caijingquan/imagebed/raw/master/1602319363_20200402113644360_1052587986.png)

Feign 支持请求拦截器，在发送请求前，可以对发送的模板进行操作，例如设置请求头等属性，自定请求拦截器需要实现 feign.RequestInterceptor 接口，该接口的方法 apply 有参数 template ，该参数类型为 RequestTemplate，我们可以根据实际情况对请求信息进行调整，示例如下：

![](https://gitee.com/caijingquan/imagebed/raw/master/1602319367_20200402113755119_206506254.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1602319368_20200402113836526_1488443642.png)

![](https://gitee.com/caijingquan/imagebed/raw/master/1602319368_20200402114022000_324177446.png)

ClientHttpRequestInterceptor
![](https://gitee.com/caijingquan/imagebed/raw/master/1602319369_20200402114240808_495076602.png)



