[TOC]

这些注解都是对@Component的扩展

# @Controller
用于标注是控制层组件，需要返回页面时请用@Controller而不是@RestController
如果不加@ResponseBody那么直接返回页面

# @ResponseBody
表示该方法的返回结果直接写入HTTP response body中，一般在异步获取数据时使用，在使用@RequestMapping后，返回值通常解析为跳转路径
加上@responsebody后返回结果不会被解析为跳转路径，而是直接写入HTTP response body中；比如异步获取json数据，加上@responsebody后，会直接返回json数据；

# @RestController
@RestController的作用相当于@Controller和@ResponseBody一起使用的结果

# @Service
约定

# @Reponsitory
约定

# @ReqyestMapping
RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径；
该注解有六个属性:
+ params:指定request中必须包含某些参数值是，才让该方法处理。
+ headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
+ value:指定请求的实际地址，指定的地址可以是URI Template 模式
+ method:指定请求的method类型， GET、POST、PUT、DELETE等
+ consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
+ produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回。

# GetMapping等等
相当于@RequestMapping（value=”/”，method=RequestMethod.GetPostPutDelete等） 。是个组合注解

# @RequestParam
用在方法的参数前面。相当于 request.getParameter；

# @PathVariable
路径变量。如 RequestMapping(“user/get/mac/{macAddress}”)

# @RequestBody

# @Valid


