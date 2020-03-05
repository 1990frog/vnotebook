generatorConfig.xml
```xml
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="generator/config.properties"/>

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
        </plugin>
        <!-- 数据库信息 -->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.url}"
                        userId="${jdbc.user}"
                        password="${jdbc.password}">
        </jdbcConnection>
        <!-- 生产entity文件目录 -->
        <javaModelGenerator targetPackage="com.source.mvc.domain.entity.${module}"
                            targetProject="src/main/java"/>
        <!-- 生成mapper.xml -->
        <sqlMapGenerator targetPackage="mapper.${module}"
                         targetProject="src/main/resources"/>
        <!-- 生成mapper接口 -->
        <javaClientGenerator targetPackage="com.source.mvc.dao.${module}"
                             targetProject="src/main/java"
                             type="XMLMAPPER"/>
        <!-- 生成上述元素的表清单 -->
        <table tableName="${table}">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
    </context>
</generatorConfiguration>
```
config.properties
```
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/springboot
jdbc.user=root
jdbc.password=123456

# 模块名称
module=user
# 表名
table=user
```