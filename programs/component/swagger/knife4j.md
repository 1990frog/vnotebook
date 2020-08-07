[TOC]

# 导入
```xml
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>knife4j-spring-boot-starter</artifactId>
	<version>2.0.4</version>
</dependency>
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>knife4j-spring-boot-autoconfigure</artifactId>
	<version>2.0.4</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
```
# 配置
```java
@Configuration
@EnableSwagger2
@EnableKnife4j
@Import(BeanValidatorPluginsConfiguration.class)
public class SwaggerConfiguration {

	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2)
				.apiInfo(apiInfo())
				.select()
//				.apis(RequestHandlerSelectors.basePackage("com"))
				.apis(RequestHandlerSelectors.withClassAnnotation(Api.class))
				.paths(PathSelectors.any())
				.build();
	}

	private ApiInfo apiInfo() {
		return new ApiInfoBuilder()
				.title("title")
				.build();
	}
}
```

# 注解
## 定义模块
```
@Api(value = "模块", tags = {"模块"})
```
## 模块排序
```
@ApiSort(1)
```
## 定义接口文档
```
@ApiOperation("接口文档")
```
## 接口增强
```
@ApiOperationSupport(order = 13, author = "author", responses=responses, ignoreParameters={}, includeParameters={})
```
+ order：排序
+ author：作者
+ response：返回类型
+ ignoreParameters：忽略掉的参数
+ includeParameters：包含的参数

# 参数描述
```
@ApiImplicitParams({
            @ApiImplicitParam(name = "param", value = "param", required = true, paramType = "query", dataType = "String")
    })
```

# 实体类
```
@ApiModel
```

# 实体类变量
```
@ApiModelProperty
```