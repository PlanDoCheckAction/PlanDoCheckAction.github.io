---
layout: post
title: Spring-（1）HelloSpring
categories: Java
description: 初识 Spring
keywords: Java, Spring
---

# Spring-（1）HelloSpring

首先介绍一下 Spring，Spring 是最受欢迎的企业级 Java 应用程序开发框架。数以百万的来自世界各地的开发人员使用 Spring 框架来创建好性能、易于测试、可重用的代码。其次，Spring 是轻量级的，Spring 框架是一个开源的 Java 平台，  Spring 框架的基础版本是在 2MB 左右的。

Spring 框架的核心特性可以用于开发任何 Java 应用程序，但是在 JavaEE 平台上构建 web 应用程序是需要扩展的。Spring 框架的目标是使 J2EE 开发变得更容易使用，通过启用基于 POJO 编程模型来促进良好的编程实践。

此文章建议有一定 JavaEE 基础的程序员学习，本人也是第一次接触 Spring，如有写得不正确的地方，希望大家踊跃指出。

先说一下 Spring 的核心基础三个概念：

- **IOC:Inversion of Control，即「控制反转」，这并不是什么技术，而是一种设计思想，控制反转的意思是将控制权交出去，例如平常我们需要一个对象一般都是 new 一个,而用了 Spring IOC 后，我们就会将 new 对象的操作交给 IOC 容器，从而省去很多麻烦，这就是将控制权交给 IOC 容器，即「控制反转」，讲到 IOC（控制反转）那一定要了解 DI（依赖注入）的使用；**

- **DI：Dependency Injection，即「依赖注入」，依赖注入可以有高效的解耦合的能力。可以说，依赖注入仅仅是控制反转的一个具体的例子，当编写一个复杂的 Java 应用程序时，应用程序类应该尽可能的独立于其他的 Java 类来增加这些类可重用的可能性，当进行单元测试时，可以使它们独立于其他类进行测试。依赖注入有助于将这些类粘合在一起，并且在同一时间让它们保持独立。我们将这两个部分拆开来理解，依赖的关系就像是类里面套着另一个类，例如车店是一个类，车店类里面有一个属性是车主类，那么车店类就依赖了车主类，这就是依赖关系，那注入的意思是车主类通过 IOC 容器注入到车店，这个地方我没有过细的讲，在以后我会再详细的说明的，目前就先做一个了解！**
 
- **AOP：Aspect Oriented Programming，意思是「面向切面编程」，我们以前了解的很多都是面向对象编程（OOP），而面向切面编程会解决很多问题，需要说明的一点是，AOP 只是 OOP 的一个补充，传统的编程都是纵向编程，而 AOP 则是进行横向编程，AOP 独立于应用程序的业务逻辑，就像 servlet 过滤器和 servlet 监听器一样，它可以处理一些每次都需要操作但又很繁琐的事情，比如日志记录、声明性事务、安全性和缓存等等。**

这就是Spring的核心基础，在这里我也只是简单的写了一点我的基本认识，如果有机会，我会再把上述内容再详细的写一遍。

-------------------
现在我们来自己动手写一个入门 Demo，首先我们创建一个Demo项目示例。

![](/images/posts/java/Spring-1-DemoCatalog.png)

我这边创建了一个 HelloSpring 的 Demo，然后我们导入相应的包，这里我们要用到 Spring 的 jar 包，Spring 的 jar 包可以到 Spring 的官网下载。

附上官网链接：

 https://spring.io/ 

 http://projects.spring.io/spring-framework/ 

 这里是需要导入的 jar 包（提示：第一个包是要自己下的，百度可以搜到。  PS：不是给百度打广告）：

 ![](/images/posts/java/Spring-1-Jar.png)

导入 jar 包后就开始写 Demo 了。

-------------------
这是上图中的 HelloSpring 的类里面的全部内容，我就不一一打了（任性，直接复制粘贴）。

```
public class HelloSpring {
	private String message;

	public String getMessage() {
		return message;
	}

	public void setMessage(String message) {
		this.message = message;
	}
	
}
```

这个是做测试用到的 main 方法，鉴于可能有人不会用 Junit，所以就用 main 方法做测试，反正也是一样杠杠的。这里用到的 ApplicationContext 暂时不需要详细了解，仅仅知道它可以读取 xml 文件即可。

```
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainAPP {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
		HelloSpring hSpring = (HelloSpring) context.getBean("helloSpring");
		System.out.println(hSpring.getMessage());
	}

}
```

**特别说明一下，这个是xml文件，是对Spring做配置用的，beans ... 尖括号里面的内容目前不需要做太多了解，但是我们需要了解bean的用法，bean的id是作为一个唯一的标识（相当于name），标识出它所对应的class类，而class就是所对应的类（此类需要写全限定名称），bean里面有一个property属性，property属性指定的是class类里面的属性，这里的property是将com.tbspring.HelloSpring类中的message属性赋值“Hello Spring !”字符串（注意：property的赋值方式是运用setter方法赋值注入（还有一种是通过构造器方法注入，用得比较少，以后可能讲到））**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
	
	<bean id="helloSpring" class="com.tbspring.HelloSpring">
		<property name="message" value="Hello Spring !"></property>
	</bean>

	
</beans>
```

-------------------
运行main方法，如果控制台打印出「Hello Spring !」则表示 Spring 入门的 Demo 就算成功了！

![](/images/posts/java/Spring-1-Print.png)
