---
title: Spring学习笔记
date: 2020-03-04 23:36:01
tags: 
- Java
- 框架
categories: java
---

## 1.1 IOC的概念和作用

### 1.1.1 控制反转-Inversion Of Contorl

​	控制反转（IOC）把创建对象的权利交给框架，是框架的重要特征，并非面向对象编程的专用术语。它包括依赖注入（Dependency Injection）和依赖查找（Dependency Lookup）。

​	ioc的作用：消减计算机程序的耦合（解除我们代码中的依赖关系）。

<!-- more -->

### 1.1.2 Spring中的基于XML的IOC配置

#### 1. bean标签:

​	bean标签:用于配置让spring创建对象，并且存入ioc容器之中
​	id 属性:对象的唯一标识。
​	class 属性:指定要创建对象的全限定类名
​	scope:指定对象的作用范围。

* ​    singleton :默认值，单例的.

* ​    prototype :多例的.

```xml
<!-- 第一种方式:使用默认无参构造函数 
	在默认情况下:它会根据默认无参构造函数来创建类对象。如果 bean 中没有默认无参构造函数，将会创建失败。--> 
<bean id="accountService" class="com.slovebarca.service.impl.AccountServiceImpl">
<!-- 第二种方式:spring 管理静态工厂-使用静态工厂的方法创建对象
	使用 StaticFactory 类中的静态方法 createAccountService 创建对象，并存入 spring 容器
	id 属性:指定 bean 的 id，用于从容器中获取
	class 属性:指定静态工厂的全限定类名 
	factory-method 属性:指定生产对象的静态方法-->
<bean id="accountService" class="com.itheima.factory.StaticFactory" 
      factory-method="createAccountService"></bean>
<!-- 第三种方式:spring 管理实例工厂-使用实例工厂的方法创建对象
	先把工厂的创建交给 spring 来管理。
	然后在使用工厂的 bean 来调用里面的方法 
	factory-bean 属性:用于指定实例工厂 bean 的 id。
	factory-method 属性:用于指定实例工厂中创建对象的方法。-->
  <bean id="instancFactory" class="com.itheima.factory.InstanceFactory"></bean>
  <bean id="accountService" factory-bean="instancFactory" 
        factory-method="createAccountService"></bean>
```

#### 2. ApplicationContext接口

​	ClassPathXmlApplicationContext:它是从类的根路径下加载配置文件 推荐使用这种。

​	FileSystemXmlApplicationContext: 它是从磁盘路径上加载配置文件，配置文件可以在磁盘的任意位置。

​	AnnotationConfigApplicationContext:当我们使用注解配置容器对象时，需要使用此类来创建 spring 容器。它用来读取注解。

```java
// 1.使用 ApplicationContext 接口，就是在获取 spring 容器
ApplicationContext ac = new ClasspathXmlApplicationContext("bean.xml"); 
// 2.根据 bean 的 id 获取对象
IAccountService accountService = (IAccountService) ac.getBean("accountService");	
IAccountService accountService = ac.getBean("accountService",IAccountService.class);
```

#### 3. 构造函数注入

```xml
<!-- 使用构造函数的方式，给 service 中的属性传值 
	要求:类中需要提供一个对应参数列表的构造函数。
	constructor-arg
		name:指定参数在构造函数中的名称
		value:它能赋的值是基本数据类型和 String 类型
		ref:它能赋的值是其他 bean 类型，也就是说，必须得是在配置文件中配置过的 bean -->
<bean id="accountService" class="com.slovebarca.service.impl.AccountServiceImpl">
  <constructor-arg name="name" value="zhangsan"></constructor-arg>
  <constructor-arg name="age" value="18"></constructor-arg>
  <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
<bean id="now" class="java.util.Date"></bean>
```

#### 4. set方法注入

```xml
<!-- 通过配置文件给 bean 中的属性传值:使用 set 方法的方式 
	property
		name:找的是类中 set 方法后面的部分 
		ref:给属性赋值是其他 bean 类型的 
		value:给属性赋值是基本数据类型和 string 类型的 -->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
  <property name="name" value="test"></property> 
  <property name="age" value="21"></property> 
  <property name="birthday" ref="now"></property>
</bean>
<bean id="now" class="java.util.Date"></bean>
```

#### 5. 注入集合属性

```xml
<!-- 注入集合数据
	List 结构的: array,list,set
	Map 结构的 map,entry,props,prop -->
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"> 
  <!-- 在注入集合数据时，只要结构相同，标签可以互换 -->
	<!-- 给数组注入数据 --> 
  <property name="myStrs">
		<set>
    	<value>AAA</value>
    	<value>BBB</value>
   	 <value>CCC</value>
		</set>
	</property>
	<!-- 注入 list 集合数据 --> 
  <property name="myList">
	<array>
	   <value>AAA</value>
       <value>BBB</value>
       <value>CCC</value>
    </array>
	</property>
	<!-- 注入 Map 数据 -->
	<property name="myMap">
    <props>
		<prop key="testA">aaa</prop>
    	<prop key="testB">bbb</prop>
		</props>
	</property>
	<!-- 注入 properties 数据 -->
  <property name="myProps"> 
    <map>
	  <entry key="testA" value="aaa"></entry> 
      <entry key="testB">
        <value>bbb</value>
	  </entry>
    </map>
  </property>
</bean>
```

### 1.1.3 Spring中的基于注解的IOC配置

```xml
<!-- 告知 spring 创建容器时要扫描的包 -->
<context:component-scan base-package="com.slovebarca"></context:component-scan>
```

#### 1. @Component

​	作用：把资源让 spring 来管理。相当于在 xml 中配置一个 bean。

​	属性：value：指定 bean 的 id。如果不指定 value 属性，默认 bean 的 id 是当前类的类名。首字母小写。

​	@Controller @Service @Repository：三个注解都是针对一个的衍生注解，他们的作用及属性都是一模一样的。

* ​    @Controller: 一般用于表现层的注解。

* ​    @Service: 一般用于业务层的注解。

* ​    @Repository: 一般用于持久层的注解。

#### 2. @Autowired

​	作用: 自动按照类型注入。当使用注解注入属性时，set 方法可以省略。它只能注入其他 bean 类型。当有多个类型匹配时，使用要注入的对象变量名称作为 bean 的 id，在 spring 容器查找，找到了也可以注入成功。找不到就报错。相当于:

#### 3. @Qualifier

​	作用：在自动按照类型注入的基础之上，再按照 Bean 的 id 注入。它在给字段注入时不能独立使用，必须和@Autowire 一起使用;但是给方法参数注入时，可以独立使用。

​	属性：value：指定 bean 的 id。

#### 4. @Resource

​	作用：直接按照 Bean 的 id 注入。它也只能注入其他 bean 类型。

​	属性：name：指定 bean 的 id。

#### 5. @Value

​	作用：注入基本数据类型和 String 类型数据的。

​	属性：value：用于指定值。

#### 6. @Scope

​	作用：指定 bean 的作用范围。

​	属性：value：指定范围的值。（singleton prototype request session globalsession）

#### 7. @Configuration

​	作用：用于指定当前类是一个 spring 配置类，当创建容器时会从该类上加载注解。获取容器时需要使用AnnotationApplicationContext(有@Configuration 注解的类.class)。

​	属性：value：用于指定配置类的字节码

#### 8. @ComponentScan

​	作用：用于指定 spring 在初始化容器时要扫描的包。

​	属性：basePackages：用于指定要扫描的包。和该注解中的 value 属性作用一样。

#### 9. @Bean

​	作用：该注解只能写在方法上，表明使用此方法创建一个对象，并且放入 spring 容器。

​	属性：name：给当前@Bean 注解方法创建的对象指定一个名称(即 bean 的 id)。

#### 10. @PropertySource

​	作用：用于加载.properties 文件中的配置。例如我们配置数据源时，可以把连接数据库的信息写到 properties 配置文件中，就可以使用此注解指定 properties 配置文件的位置。

​	属性：value[ ]:用于指定 properties 文件位置。如果是在类路径下，需要写上 classpath:

#### 11. @Import

​	作用：用于导入其他配置类，在引入其他配置类时，可以不用再写@Configuration 注解。当然，写上也没问题。

​	属性：value[ ]:用于指定其他配置类的字节码。

```java
//注解示例
@Configuration
@ComponentScan("com.slovebarca")
@Import({JdbcConfig.class})
@PropertySource("classpath:jdbcConfig.properties")
public class SpringConfiguration {
		
}
```

```java
public class JdbcConfig {
  
	@Value("${jdbc.driver}")
  	private String driver;
  	@Vaule("${jdbc.url}")
  	private String url;
 	@Value("${jdbc.username}") 
 	private String username;
	@Value("${jdbc.password}") 
  	private String password;
  
	@Bean(name="dataSource")
  	public DataSource createDataSource() {
    	try {
			ComboPooledDataSource ds = new ComboPooledDataSource(); 
        	ds.setDriverClass(driver);
			ds.setJdbcUrl(url);
			ds.setUser(username); 
        	ds.setPassword(password); 
            return ds;
		}catch (Exception e) {
			throw new RuntimeException(e);
		}
  	}
}
```

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_study
jdbc.username=root
jdbc.password=root
```

#### 12. Spring整合Junit

```java
//使用@RunWith 注解替换原有运行器
@RunWith(SpringJUnit4ClassRunner.class)
//使用@ContextConfiguration 指定 spring 配置文件的位置
//locations 属性:用于指定配置文件的位置。如果是类路径下，需要用classpath:表明 
//classes 属性:用于指定注解的类。当不使用 xml 配置时，需要用此属性指定注解类的位置。
@ContextConfiguration(location={"classpath:bean.xml"})
public class AccountServiceTest{
	//使用@Autowired 给测试类中的变量注入数据
  	@Autowired
  	private IAccountService as;
}
```

## 1.2 AOP的概念与作用

### 1.2.1 AOP概述

#### 1. AOP：

​	全称是 Aspect Oriented Programming 即：面向切面编程。简单的说它就是把我们程序重复的代码抽取出来，在需要执行的时候，使用动态代理的技术，在不修改源码的 基础上，对我们的已有方法进行增强。

#### 2. AOP相关术语：

​	Joinpoint（连接点 ）：

​		所谓连接点是指那些被拦截到的点。在 spring 中,这些点指的是方法,因为 spring 只支持方法类型的 连接点。 

​	Pointcut（切入点 ）：

​		所谓切入点是指我们要对哪些 Joinpoint 进行拦截的定义。

​	Advice（通知/增强）: 

​		所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知。 通知的类型:前置通知,后置通知,异常通知,最终通知,环绕通知。

​	 Introduction（引介 ）：

​		引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方 法或 Field。

​	Target（目标对象）：

​		代理的目标对象。 

​	Weaving（织入 ）：

​		是指把增强应用到目标对象来创建新的代理对象的过程。 spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。

​	 Proxy（代理） ：

​		一个类被 AOP 织入增强后，就产生一个结果代理类。

 	Aspect（切面）: 

​		是切入点和通知(引介)的结合。

### 1.2.2 spring中基于XML的AOP

#### 1. 声明 aop 配置

```xml
<aop:config></aop:config>
```

#### 2. 配置切面

```xml
<!-- id : 给切面提供一个唯一标识。
	 ref: 引用配置好的通知类 bean 的 id。 -->
<aop:aspect id="txAdvice" ref="txManager"></aop:aspect>
```

#### 3. 配置切入点表达式

```xml
<!-- expression: 用于定义切入点表达式。
	 id: 用于给切入点表达式提供一个唯一标识 -->
<aop:pointcut expression="execution(
public void com.itheima.service.impl.AccountServiceImpl.transfer(java.lang.String, java.lang.String, java.lang.Float))" id="pt1"/>
```

​	通常情况下，我们都是对业务层的方法进行增强，所以切入点表达式都是切到业务层实现类。

```
execution(* com.itheima.service.impl.*.*(..))
```

#### 4. 配置对应的通知类型

```xml
<!-- method:用于指定通知类中的增强方法名称 
	 ponitcut-ref:用于指定切入点的表达式的引用 
	 poinitcut:用于指定切入点表达式 -->
<aop:before method="beginTransaction" pointcut-ref="pt1"/>
<aop:after-returning method="commit" pointcut-ref="pt1"/>
<aop:after-throwing method="rollback" pointcut-ref="pt1"/>
<aop:after method="release" pointcut-ref="pt1"/>
```

#### 5. 环绕通知

```xml
<aop:around method="transactionAround" pointcut-ref="pt1"/>
```

```java
//spring 框架为我们提供了一个接口:ProceedingJoinPoint，它可以作为环绕通知的方法参数。
//在环绕通知执行时，spring 框架会为我们提供该接口的实现类对象，我们直接使用就行。
public Object transactionAround(ProceedingJoinPoint pjp) {
		//定义返回值
		Object rtValue = null;
		try {
			//获取方法执行所需的参数
			Object[] args = pjp.getArgs(); //前置通知:开启事务
			beginTransaction();
			//执行方法
			rtValue = pjp.proceed(args); //后置通知:提交事务
			commit();
		}catch(Throwable e) { //异常通知:回滚事务 
      		rollback(); 
      		e.printStackTrace();
		}finally { //最终通知:释放资源 release();
		}
		return rtValue;
}
```



### 1.2.3 基于注解的AOP配置

#### 在 spring 配置文件中开启 spring 对注解 AOP 的支持

```xml
<aop:aspectj-autoproxy/>
```

#### 不使用xml的配置方式

```java
@EnableAspectJAutoProxy
```

#### 1. 在通知类上使用@Aspect 注解声明为切面

​	作用：把当前类声明为切面

```java
@Component("txManager")
@Aspect
public class TransactionManager {}
```

#### 2. 在增强的方法上使用注解配置通知

​	@Before @AfterReturning @AfterThrowing @After

```java
@Before("execution(* com.itheima.service.impl.*.*(..))")
@AfterReturning("execution(* com.itheima.service.impl.*.*(..))")
@AfterThrowing("execution(* com.itheima.service.impl.*.*(..))")
@After("execution(* com.itheima.service.impl.*.*(..))")
```

#### 3. 环绕通知注解配置

```java
@Around("execution(* com.itheima.service.impl.*.*(..))")
```

#### 4. 切入点表达式注解

```java
@Pointcut("execution(* com.itheima.service.impl.*.*(..))") 
private void pt1() {}
-----------------------------------------------------------
@Around("pt1()")
```

## 1.3 Spring中的事务控制

### 1.3.1 Spring 中事务控制的 API 介绍

#### 1. PlatformTransactionManager

​	此接口是 spring 的事务管理器，它里面提供了我们常用的操作事务的方法。

```java
// 获取事务状态信息
TransactionStatus getTransaction(TransactionDefinition definition);
// 提交事务
void commit(TransactionStatus status);
// 回滚事务
void rollback(TransactionStatus status);
```

​	真正管理事务的对象 org.springframework.jdbc.datasource.DataSourceTransactionManager，使用 Spring JDBC 或 iBatis 进行持久化数据时使用

#### 2. TransactionDefinition

​	它是事务的定义信息对象,里面有如下方法:

```java
// 获取事务名称对象
String getName();
// 获取事务隔离级别
int getIsolationLevel();
// 获取事务传播行为（默认值：REQUIRED）
int getPropagationBehavior();
// 获取事务超时时间（默认-1）
int getTimeout();
// 获取事务是否只读（建议查询时设为只读）
boolean isReadOnly();
```

#### 3. TransactionStatus

​	此接口提供的是事务具体的运行状态

### 1.3.2 基于 XML 的声明式事务控制

#### 1. 配置事务管理器

```xml
<bean id="transactionManager"  class="org.springframework.jdbc.datasource.	DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

#### 2. 配置事务的通知引用事务管理器

```xml
<tx:advice id="txAdvice" transaction-manager="transactionManager"> </tx:advice>
```

#### 3. 配置事务的属性

```xml
<!--在 tx:advice 标签内部 配置事务的属性 -->
<tx:attributes>
<!-- 指定方法名称:是业务核心方法
	read-only:是否是只读事务。默认 false，不只读。 
	isolation:指定事务的隔离级别。默认值是使用数据库的默认隔离级别。
	propagation:指定事务的传播行为。
	timeout:指定超时时间。默认值为:-1。永不超时。
	rollback-for:用于指定一个异常，当执行产生该异常时，事务回滚。产生其他异常 ，事务不回滚。没有默认值，任何异常都回滚。
	no-rollback-for:用于指定一个异常，当产生该异常时，事务不回滚，产生其他异常时，事务回滚。没有默认值，任何异常都回滚。-->
	<tx:method name="*" read-only="false" propagation="REQUIRED"/> 
  <tx:method name="find*" read-only="true" propagation="SUPPORTS"/>
</tx:attributes>
```

#### 4. 配置 AOP 切入点表达式

```xml
<aop:config>
	<aop:pointcut expression="execution(* com.itheima.service.impl.*.*(..))" id="pt1"/> </aop:config>
```

#### 5. 配置切入点表达式和事务通知的对应关系

```xml
<!-- 在 aop:config 标签内部:建立事务的通知和切入点表达式的关系 -->
<aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
```

### 1.3.3 基于注解的声明式事务控制

#### 在配置文件中开启 spring 对注解事务的支持

```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```v

#### 不使用xml的配置方式

```java
@EnableTransactionManagement
```

####  在业务层使用@Transactional 注解

```java
@Transactional(readOnly=true,propagation=Propagation.SUPPORTS)
```

