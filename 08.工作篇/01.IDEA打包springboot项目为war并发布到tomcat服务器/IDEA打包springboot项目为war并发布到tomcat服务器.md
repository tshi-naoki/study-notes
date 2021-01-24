# spring boot 打成war

## springboot项目打包配置

首先在启动类目录下新增打包类

启动类继承`SpringBootServletInitializer`覆写configure方法，作用与web项目中的web.xml一样。

```java
@SpringBootApplication
public class AnswerManagerApplication extends SpringBootServletInitializer {
    public static void main(String[] args) {
        SpringApplication.run(AnswerManagerApplication.class, args);
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(AnswerManagerApplication.class);
    }
}
```

> 启动类继承自SpringBootServletInitializer方可正常部署至常规tomcat下，其主要能够起到web.xml的作用（web.xml主要配置各种servlet，filter，listener等，如常见的Log4jConfigListener、OpenSessionInViewFilter、CharacterEncodingFilter、DispatcherServlet等，此部分信息均是容器启动时加载）

## pom.xml文件改动

需要把打包方式从jar包改成war

```xml
<packaging>war</packaging>
```

修改jdk版本和服务器的一致

```xml
<properties>
    <java.version>8</java.version>
</properties>
```

# websocket配置

参考：https://blog.csdn.net/whiteWYC/article/details/83094422

项目中使用到了websocket，在发布到服务器时，是不需要注入到spring容器的，所以不需要配置WebSocketConfig。

创建包`websocket`，创建Java文件`WebSocketServer`，添加注解

```java
@ServerEndpoint(value = "/webSocket/{sid}")
@Component
```

正常编写websocket就好。

# Tomcat设置默认项目和端口

正常部署，在不作为默认项目的时候没有问题，tomcat正常启动，但是如果通过

```xml
<Context path="" docBase="/answer-manager" debug="0" reloadable="false"/>
```

配置为默认项目，会出现资源加载失败的情况，浏览器报503，表示有服务器启动了，但是访问的这个网址解析不了，找不到网页信息返回。查看tomcat目录下的log日志目录：

catalina.xxxxx日期的文件中：

```shell
23-Jan-2021 16:23:58.093 SEVERE [main] org.apache.catalina.core.ContainerBase.startInternal A child container failed during start
 java.util.concurrent.ExecutionException: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[localhost]]
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:939)
	at org.apache.catalina.core.StandardEngine.startInternal(StandardEngine.java:262)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	at org.apache.catalina.core.StandardService.startInternal(StandardService.java:422)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	at org.apache.catalina.core.StandardServer.startInternal(StandardServer.java:793)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	at org.apache.catalina.startup.Catalina.start(Catalina.java:655)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.catalina.startup.Bootstrap.start(Bootstrap.java:355)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.commons.daemon.support.DaemonLoader.start(DaemonLoader.java:243)
Caused by: org.apache.catalina.LifecycleException: Failed to start component [StandardEngine[Catalina].StandardHost[localhost]]
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:167)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1419)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1409)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: org.apache.catalina.LifecycleException: A child container failed during start
	at org.apache.catalina.core.ContainerBase.startInternal(ContainerBase.java:947)
	at org.apache.catalina.core.StandardHost.startInternal(StandardHost.java:872)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	... 6 more
```

`A child container failed during start`：启动子容器失败。

**在配置tomcat默认项目的时候需要配置绝对路径，才能让tomcat启动的时候找到该项目**，但是奇怪的是，我在本地tomcat配置默认项目的时候，使用的是相对路径，可以访问。