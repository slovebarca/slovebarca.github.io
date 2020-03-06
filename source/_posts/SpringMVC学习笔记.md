---
title: SpringMVC学习笔记
date: 2020-03-05 03:29:59
tags: 
- java
- 框架
categories: java
---
## 1.1 关于三层架构和 MVC
### 1.1.1 三层架构
#### 1. 表现层
&emsp;&emsp;也就是我们常说的 web 层。它负责接收客户端请求，向客户端响应结果，通常客户端使用 http 协议请求 web 层，web 需要接收 http 请求，完成 http 响应。
<!-- more -->
&emsp;&emsp;表现层包括展示层和控制层:控制层负责接收请求，展示层负责结果的展示。
&emsp;&emsp;表现层依赖业务层，接收到客户端请求一般会调用业务层进行业务处理，并将处理结果响应给客户端。 表现层的设计一般都使用 MVC 模型。(MVC 是表现层的设计模型，和其他层没有关系).
#### 2. 业务层
&emsp;&emsp;也就是我们常说的 service 层。它负责业务逻辑处理，和我们开发项目的需求息息相关。web 层依赖业 务层，但是业务层不依赖 web 层。
&emsp;&emsp;业务层在业务处理时可能会依赖持久层，如果要对数据持久化需要保证事务一致性。(也就是我们说的， 事务应该放到业务层来控制)。
#### 3. 持久层
&emsp;&emsp;也就是我们是常说的 dao 层。负责数据持久化，包括数据层即数据库和数据访问层，数据库是对数据进 行持久化的载体，数据访问层是业务层和持久层交互的接口，业务层需要通过数据访问层将数据持久化到数据库中。通俗的讲，持久层就是和数据库交互，对数据库表进行曾删改查的。
### 1.1.2 MVC模型
#### 1. Model(模型): 
&emsp;&emsp;通常指的就是我们的数据模型。作用一般情况下用于封装数据。
#### 2. View(视图):
&emsp;&emsp;通常指的就是我们的 jsp 或者 html。作用一般就是展示数据的。 通常视图是依据模型数据创建的。
#### 3. Controller(控制器): 
&emsp;&emsp;是应用程序中处理用户交互的部分。作用一般就是处理程序逻辑的。
### 1.2 SpringMVC概述
&emsp;&emsp;SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架。
![position](http://q6opupbjc.bkt.clouddn.com/springmvc1.png)
## 2.1 SpringMVC入门
### 2.1.1 入门案例
#### 1. 配置核心控制器
```xml
<servlet>
    <servlet-name>SpringMVCDispatcherServlet</servlet-name> 
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <!-- 配置初始化参数，用于读取 SpringMVC 的配置文件 --> 
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:SpringMVC.xml</param-value> 
    </init-param>
    <!-- 配置 servlet 的对象的创建时间点:应用加载时创建。-->
    <load-on-startup>1</load-on-startup> 
</servlet>
```
#### 2. 创建SpringMVC配置文件
```xml
<!-- 配置创建 spring 容器要扫描的包 -->
<context:component-scan base-package="com.itheima"/>
<!-- 配置视图解析器 -->
<bean id="internalResourceViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>
<!-- 注解支持 -->
<mvc:annotation-driven/>
```
#### 3. 编写控制器并使用注解配置
```java
@Controller("helloController")
public class HelloController {
    @RequestMapping("/hello") 
    public String sayHello() {
        System.out.println("HelloController 的 sayHello 方法执行了。。。。");
        return "success"; 
    }
}
```
#### 4. 案例执行过程
- 服务器启动，应用被加载。读取到 web.xml 中的配置创建 spring 容器并且初始化容器中的对象。
- 浏览器发送请求，被 DispatherServlet 捕获，该 Servlet 并不处理请求，而是把请求转发出去。转发 的路径是根据请求 URL，匹配@RequestMapping 中的内容。
- 匹配到了后，执行对应方法。该方法有一个返回值。
- 根据方法的返回值，借助 InternalResourceViewResolver 找到对应的结果视图。 
- 渲染结果视图，响应浏览器。
### 2.1.2 入门案例中涉及的组件
#### 1. DispatcherServlet:前端控制器
&emsp;&emsp;用户请求到达前端控制器，它就相当于 mvc 模式中的 c，dispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性。
#### 2. HandlerMapping:处理器映射器
&emsp;&emsp;HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的 映射方式，例如:配置文件方式，实现接口方式，注解方式等。
#### 3. Handler:处理器
&emsp;&emsp; 它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理。
#### 4. View Resolver:视图解析器
&emsp;&emsp; View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。
#### 5. View:视图
&emsp;&emsp; 一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开 发具体的页面。
### 2.1.3 RequestMapping 注解
#### 1. 属性
- value:用于指定请求的 URL。它和 path 属性的作用是一样的。
- method:用于指定请求的方式。（GET/POST)
- params:用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的 key 和 value 必须和 配置的一模一样。
### 2.2.1 请求参数乱码
```xml
<!-- 配置 springMVC 编码过滤器 --> 
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class> org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!-- 设置过滤器中的属性值 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value> 
    </init-param>
    <!-- 启动过滤器 --> 
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value> 
    </init-param>
</filter>
<!-- 过滤所有请求 --> 
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```
