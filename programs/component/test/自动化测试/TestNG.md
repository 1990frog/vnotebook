[TOC]

# maven导入
```
<!-- https://mvnrepository.com/artifact/org.testng/testng -->
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.1.0</version>
    <scope>test</scope>
</dependency>
```

# testng VS junit

# TestNG
@Test
@BeforeMethod
@AfterMethod
@BeforeClass
@AfterClass
@BeforeSuite
@AfterSuite

# 套件测试
配置文件
```xml
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="suite">
    <test name="login">
        <classes>
            <class name="com.testng.SuiteConfig"/>
            <class name="com.testng.LoginController"/>
        </classes>
    </test>
</suite>
```
运行：
执行run配置文件

# 忽略测试
```java
public class IgnoreTest {

    @Test
    public void ignoreMethod1(){
        System.out.println("ignoremethod1执行");
    }

//    @Test(enabled = false)
    @Test
    public void ignoreMethod2(){
        System.out.println("ignoremethod2执行");
    }
}
```
run IgnoreTest
两个方法都被执行
```java
public class IgnoreTest {

    @Test
    public void ignoreMethod1(){
        System.out.println("ignoremethod1执行");
    }

    @Test(enabled = false)
    public void ignoreMethod2(){
        System.out.println("ignoremethod2执行");
    }
}
```
run IgnoreTest
只有方法1执行了

# 组测试
## 方法分组
```java
public class GroupOnMethod {

    @Test(groups = "server")
    public void server1(){
        System.out.println("服务端组1");
    }

    @Test(groups = "server")
    public void server2(){
        System.out.println("服务端组2");
    }

    @Test(groups = "client")
    public void client1(){
        System.out.println("客户端组1");
    }

    @Test(groups = "client")
    public void client2(){
        System.out.println("客户端组2");
    }

    @BeforeGroups("server")
    public void beforeGroupsOnServer(){
        System.out.println("这是服务端组运行之前运行的方法");
    }

    @AfterGroups("server")
    public void afterGroupsOnServer(){
        System.out.println("这是服务端组运行之后运行的方法");
    }
}
```
## 类分组
```java
@Test(groups = "classA")
public class GroupsOnClass1 {

    public void test1(){
        System.out.println("1-1");
    }

    public void test2(){
        System.out.println("1-2");
    }
}
@Test(groups = "classA")
public class GroupsOnClass2 {

    public void test1(){
        System.out.println("2-1");
    }

    public void test2(){
        System.out.println("2-2");
    }
}
@Test(groups = "classB")
public class GroupsOnClass3 {

    public void test1(){
        System.out.println("3-1");
    }

    public void test2(){
        System.out.println("3-2");
    }
}
```
配置文件
```xml
<suite name="groups">
    <test name="runAll">
        <classes>
            <class name="com.testng.group.clazz.GroupsOnClass1"></class>
            <class name="com.testng.group.clazz.GroupsOnClass2"></class>
            <class name="com.testng.group.clazz.GroupsOnClass3"></class>
        </classes>
    </test>
    <test name="runGroup">
        <groups>
            <run>
                <include name="classA"></include>
            </run>
        </groups>
        <classes>
            <class name="com.testng.group.clazz.GroupsOnClass1"></class>
            <class name="com.testng.group.clazz.GroupsOnClass2"></class>
            <class name="com.testng.group.clazz.GroupsOnClass3"></class>
        </classes>
    </test>
</suite>
```

