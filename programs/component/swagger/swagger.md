[TOC]

# @Api()
用在请求的类上，表示对类的说明，也代表了这个类是swagger2的资源
```
参数：
tags：说明该类的作用，参数是个数组，可以填多个。
value="该参数没什么意义，在UI界面上不显示，所以不用配置"
description = "用户基本信息操作"
```
# @ApiOperation()
用于方法，表示一个http请求访问该方法的操作
```
参数： 
value="方法的用途和作用"    
notes="方法的注意事项和备注"    
tags：说明该方法的作用，参数是个数组，可以填多个。
格式：tags={"作用1","作用2"} 
（在这里建议不使用这个参数，会使界面看上去有点乱，前两个常用）
```
# @ApiModel()
用于响应实体类上，用于说明实体作用
```
参数：
description="描述实体的作用"  
```
# @ApiModelProperty
用在属性上，描述实体类的属性
```
参数：
value="用户名"  描述参数的意义
name="name"    参数的变量名
required=true     参数是否必选
```
# @ApiImplicitParams
用在请求的方法上，包含多@ApiImplicitParam
# @ApiImplicitParam
用于方法，表示单独的请求参数
```
参数：
name="参数ming" 
value="参数说明" 
dataType="数据类型" 
paramType="query" 表示参数放在哪里
    · header 请求参数的获取：@RequestHeader
    · query   请求参数的获取：@RequestParam
    · path（用于restful接口） 请求参数的获取：@PathVariable
    · body（不常用）
    · form（不常用） 
defaultValue="参数的默认值"
required="true" 表示参数是否必须传
```
# @ApiParam()
用于方法，参数，字段说明 表示对参数的要求和说明
```
参数：
name="参数名称"
value="参数的简要说明"
defaultValue="参数默认值"
required="true" 表示属性是否必填，默认为false
```
# @ApiResponses
用于请求的方法上，根据响应码表示不同响应
一个@ApiResponses包含多个@ApiResponse
# @ApiResponse
用在请求的方法上，表示不同的响应
```
参数：
code="404"    表示响应码(int型)，可自定义
message="状态码对应的响应信息"
```
# @ApiIgnore()
用于类或者方法上，不被显示在页面上
# @Profile({“dev”, “test”})
用于配置类上，表示只对开发和测试环境有用

