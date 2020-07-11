---
layout: post
title: Spring-（5）Spring AOP
categories: [Java, Spring]
description: Spring AOP
keywords: Java, Spring
---

# Spring-（5）Spring AOP
AOP：Aspect Oriented Programming，意思是「面向切面编程」，我们以前了解的很多都是面向对象编程（OOP），而面向切面编程会解决很多问题，需要说明的一点是，AOP 只是 OOP 的一个补充，传统的编程都是纵向编程，而 AOP 则是进行横向编程，AOP 独立于应用程序的业务逻辑，就像 servlet 过滤器和 servlet 监听器一样，它可以处理一些每次都需要操作但又很繁琐的事情，比如日志记录、声明性事务、安全性和缓存等等。

Spring 框架的一个关键组件是面向切面的编程（AOP）框架。面向切面的编程需要把程序逻辑分解成不同的部分称为所谓的关注点。跨一个应用程序的多个点的功能被称为横切关注点，这些横切关注点在概念上独立于应用程序的业务逻辑。

在 OOP 中，关键单元模块度是类，而在 AOP 中单元模块度是切面。依赖注入帮助你对应用程序对象相互解耦和而 AOP 可以帮助你从它们所影响的对象中对横切关注点解耦。

Spring AOP 模块提供拦截器来拦截一个应用程序，例如，当执行一个方法时，你可以在方法执行之前或之后添加额外的功能。

------------------------------

AOP术语：

 - Aspect：一个模块具有一组提供横切需求的 APIs。例如，一个日志模块为了记录日志将被 AOP 方面调用。应用程序可以拥有任意数量的方面，这取决于需求。

 - Join point：在你的应用程序中它代表一个点，你可以在插件 AOP 方面。你也能说，它是在实际的应用程序中，其中一个操作将使用 Spring AOP 框架。

 - Advice：这是实际行动之前或之后执行的方法。这是在程序执行期间通过 Spring AOP 框架实际被调用的代码。

 - Pointcut：这是一组一个或多个连接点，通知应该被执行。你可以使用表达式或模式指定切入点正如我们将在 AOP 的例子中看到的。

 - Introduction：引用允许你添加新方法或属性到现有的类中。

 - Target object：被一个或者多个方面所通知的对象，这个对象永远是一个被代理对象，也称为被通知对象。

 - Weaving：Weaving 把方面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象，这些可以在编译时，类加载时和运行时完成。

Spring AOP 通知的类型：

1. 前置通知：在一个方法执行之前，执行通知；

2. 后置通知：在一个方法执行之后，不考虑其结果，执行通知；

3. 返回后通知：在一个方法执行之后，只有在方法成功完成时，才能执行通知；

4. 抛出异常后通知：	在一个方法执行之后，只有在方法退出抛出异常时，才能执行通知；

5. 环绕通知：在建议方法调用之前和之后，执行通知。

-----------------------------------
Spring 支持 @AspectJ annotation style 的方法和基于模式的方法来实现自定义方面。

我们分别来写一个关于日志记录的例子，首先我们需要导入 Spring 的 jar 包和一些第三方 jar 包：

	-org.springframework.aop-3.1.1.RELEASE.jar(在Spring 3.0之后Spring官网所提供的jar包)
	-aspectjrt.jar
	-aspectjweaver.jar
	-aopalliance.jar

用到 Spring AOP 必须得在 beans 标签内加入以下内容：

```

xmlns:aop="http://www.springframework.org/schema/aop"

http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
```

## Spring 中基于 AOP 的 XML 架构

xml 文件的配置：

```
<aop:config>
    	<aop:aspect id="log" ref="logging">
    	 <aop:pointcut id="selectAll" 
         				expression="execution(* com.tbspringAOP.schema.*.*(..))"/><!-- 定义一个pointcut -->
         <aop:before pointcut-ref="selectAll" method="beforeAdvice"/>
         <aop:after pointcut-ref="selectAll" method="afterAdvice"/>
         <aop:after-returning pointcut-ref="selectAll" 
                             	 returning="retVal"
                            	  method="afterReturningAdvice"/>
         <aop:after-throwing pointcut-ref="selectAll" 
                          	   throwing="ex"
                          	    method="AfterThrowingAdvice"/>
    	</aop:aspect>
    </aop:config>
    
    <bean id="logging" class="com.tbspringAOP.schema.Logging" ></bean>
    
    <bean id="student" class="com.tbspringAOP.schema.Student">
    	<property name="name" value="涂bin"></property>
    	<property name="age" value="19"></property>
    </bean>
```

Logging 类：

```
public class Logging {

	public void beforeAdvice(){
	      System.out.println("Going to setup student profile.");
	}
	
	public void afterAdvice(){
	      System.out.println("Student profile has been setup.");
	}
	
	public void afterReturningAdvice(Object retVal){
	      System.out.println("Returning:" + retVal.toString() );
	}
	
	public void AfterThrowingAdvice(IllegalArgumentException ex){
	      System.out.println("There has been an exception: " + ex.toString());   
	}
	
}
```

Student 类:

```
public class Student {
	private String name;
	private String age;
	public String getName() {
		System.out.println("Name : " + name );
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAge() {
		System.out.println("Age : " + age );
		return age;
	}
	public void setAge(String age) {
		this.age = age;
	}
	public void printThrowException(){
	       System.out.println("Exception raised");
	       throw new IllegalArgumentException();
	}
}
```

测试 Main：

```
public class MainAPP {

	public static void main(String[] args) {
		ApplicationContext context = 
	             new ClassPathXmlApplicationContext("Beans.xml");
	      Student student = (Student) context.getBean("student");
	      student.getName();
	      student.getAge();      
	      student.printThrowException();
	}

}
```

控制台输出：

![](..\images\posts\java\Spring-5-Print1.png)

------------------------------------
## Spring 中基于 AOP 的 @AspectJ

xml 文件的配置：

```
<!-- 如果要使用注解必须在beans标签之间加上下面一句话 -->
<aop:aspectj-autoproxy/>

<!-- 将所需类注入到IOC容器中 -->
<bean id="logging2" class="com.tbspringAOP.annotation.Logging" ></bean>
    
    <bean id="student2" class="com.tbspringAOP.annotation.Student">
    	<property name="name" value="涂涂"></property>
    	<property name="age" value="18"></property>
    </bean>
```

Logging 类：

```
//声明一个aspect
@Aspect
public class Logging {
	//声明一个切入点，execution括号里面的表示作用的范围
	@Pointcut("execution(* com.tbspringAOP.annotation.*.*(..))")
	private void selectAll(){}
	
	//在方法调用之前执行
	@Before("selectAll()")
	public void beforeAdvice(){
	      System.out.println("Going to setup student profile.");
	}
	
	//在方法调用之后执行（无论抛不抛异常）
	@After("selectAll()")
	public void afterAdvice(){
	      System.out.println("Student profile has been setup.");
	}
	

	//在方法成功调用之后执行
	@AfterReturning(pointcut="selectAll()",returning="retVal")
	public void afterReturningAdvice(Object retVal){
	      System.out.println("Returning:" + retVal.toString() );
	}
	
	//在方法抛出异常时执行
	@AfterThrowing(pointcut="selectAll()",throwing="ex")
	public void AfterThrowingAdvice(IllegalArgumentException ex){
	      System.out.println("There has been an exception: " + ex.toString());   
	}
	
}
```

Student 类：

```
public class Student {
	private String name;
	private String age;
	public String getName() {
		System.out.println("Name : " + name );
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getAge() {
		System.out.println("Age : " + age );
		return age;
	}
	public void setAge(String age) {
		this.age = age;
	}
	public void printThrowException(){
	    System.out.println("Exception raised");
	    throw new IllegalArgumentException();
	}
}
```

测试 Main：

```
public class MainAPP {

	public static void main(String[] args) {
		ApplicationContext context = 
	             new ClassPathXmlApplicationContext("Beans.xml");
	      Student student = (Student) context.getBean("student2");
	      student.getName();
	      student.getAge();      
	      student.printThrowException();
	}

}
```

最后控制台输出：

![](..\images\posts\java\Spring-5-Print2.png)

------------------------------------

对于环绕通知这里没有具体提及，有兴趣的小伙伴可以自己研究研究！

本人对 AOP 的理解并不是很深，所以目前很多内容都是借鉴他人的，不对之处，希望大家踊跃提出！
