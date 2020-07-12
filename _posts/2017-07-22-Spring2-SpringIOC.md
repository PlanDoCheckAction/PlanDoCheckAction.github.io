---
layout: post
title: Spring-（2）Spring IOC
categories: Spring
description: Spring IOC
keywords: Java, Spring
---

# Spring-（2）Spring IOC

IOC—Inversion of Control，即「控制反转」，不是什么技术，而是一种设计思想。在 Java 开发中，IOC 意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

那如何理解好 IOC 呢？

理解好 IOC 的关键是要明确「谁控制谁，控制什么」，「为何是反转（有反转就应该有正转了），哪些方面反转了」，那我们来深入分析一下：

- 谁控制谁，控制什么：传统 JavaSE 程序设计，我们直接在对象内部通过 new 进行创建对象，是程序主动去创建依赖对象；而 IOC 是有专门一个容器来创建这些对象，即由 IOC 容器来控制对象的创建；谁控制谁？当然是 IOC 容器控制了对象；控制什么？那就是主要控制了外部资源获取（不只是对象包括比如文件等）。

- 为何是反转，哪些方面反转了：有反转就有正转，传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。

----------------------------
这是 Spring 官方文档上对 Spring IOC 容器的描述：

![](/images/posts/java/Spring-2-IOC.png)

这个意思是将 Java 的 POJO 类注入到 Spring 的 IOC 容器中，然后容器利用 Java 的 POJO 类和配置元数据来生成完全配置和可执行的系统或应用程序，如果我们需要一个对象，我们只需要告诉 IOC 容器，然后 IOC 容器就会帮我 new 一个对象，这就是 IOC 容器的作用。

## Spring 提供了以下两种不同类型的容器

- Spring BeanFactory 容器：它是最简单的容器，给 DI 提供了基本的支持，它用 org.springframework.beans.factory.BeanFactory 接口来定义。BeanFactory 或者相关的接口，如 BeanFactoryAware，InitializingBean，DisposableBean，在 Spring 中仍然存在具有大量的与 Spring 整合的第三方框架的反向兼容性的目的。

- Spring ApplicationContext 容器：该容器添加了更多的企业特定的功能，例如从一个属性文件中解析文本信息的能力，发布应用程序事件给感兴趣的事件监听器的能力。该容器是由 org.springframework.context.ApplicationContext 接口定义。

	需要说明的是，ApplicationContext 的功能比 BeanFactory 更多，它包含了 BeanFactory，所以一般我们做大型来说，用 ApplicationContext 更好。但 BeanFactory 仍然可以用于轻量级的应用程序，如移动设备或基于 applet 的应用程序，其中它的数据量和速度是显著。

### Spring BeanFactory 容器

这是一个最简单的容器，它主要的功能是为依赖注入（DI）提供支持，这个容器接口在 org.springframework.beans.factory.BeanFactor 中被定义。BeanFactory 和相关的接口，比如：BeanFactoryAware、 DisposableBean、InitializingBean，仍旧保留在 Spring 中，主要目的是向后兼容已经存在的和那些 Spring 整合在一起的第三方框架。

在 Spring 中，有大量对 BeanFactory 接口的实现。其中，最常被使用的是 XmlBeanFactory 类。这个容器从一个 XML 文件中读取配置元数据，由这些元数据来生成一个被配置化的系统或者应用。

在资源宝贵的移动设备或者基于 applet 的应用当中，BeanFactory 会被优先选择。否则，一般使用的是 ApplicationContext，除非你有更好的理由选择 BeanFactory。

我们来看一个例子：

![](/images/posts/java/Spring-2-DemoCatalog1.png)

HelloSpring是一个POJO类（直接贴源代码）：

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

Bean.xml(关于基本的配置我已经在 [Spring-（1）HelloSpring](/2017/07/21/Spring1-HelloSpring) 中讲到了，废话不多说，直接贴代码)

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
		<bean id="springBeanFactory" class="com.tbspringIOC.BeanFactory.HelloSpring">
		<property name="message" value="Hello Spring BeanFactory !"></property>
	</bean>
</beans>
```

最后再把测试的 Main 贴上来（呐，注释我都写好了，我真是尽心尽力！）

PS：getBean 方法里面的参数是 xml 文件里面 bean 的 id 值。

```
public class MainAPP {

	public static void main(String[] args) {
		//利用框架提供的 XmlBeanFactory() API 去生成工厂 bean 以及利用 ClassPathResource() API 去加载在路径 CLASSPATH 下可用的 bean 配置文件
		BeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("Beans.xml"));
		//用beanFactory的getBean方法也可以获得bean对象（一般都用ApplicationContext，ApplicationContext比BeanFactory功能更全）
		//BeanFactory 仍然可以在轻量级应用中使用，比如移动设备或者基于 applet 的应用程序。
		HelloSpring hSpring = (HelloSpring) beanFactory.getBean("springBeanFactory");
		System.out.println(hSpring.getMessage());
	}

}
```

然后打印结果成功！

![](/images/posts/java/Spring-2-Print.png)

Spring BeanFactory 容器到这里就差不多结束了！

-------------------------------------
### Spring ApplicationContext 容器
Spring ApplicationContext 容器也很简单，我们在 Spring-（1）HelloSpring 的例子中用的就是 ApplicationContext（节约时间，废话不多说，直接贴代码（其实是自己懒不想打字！））。

![](/images/posts/java/Spring-2-DemoCatalog2.png)

看一下目录结构，HelloSpring 类，Beans.xml 与上面用到的例子是一样的，我们直接看一下 Main 中不一样的部分。

```
public class MainAPP {

	public static void main(String[] args) {
		//我们把BeanFactory改成了ApplicationContext
		ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
		HelloSpring hSpring = (HelloSpring) context.getBean("springApplicationContext");
		System.out.println(hSpring.getMessage());
	}

}
```

这就是 Spring ApplicationContext 容器

----------------------------------
### 现在来看一下 Bean 的用法

我们在上面的例子中已经接触了 Bean 中的 id 和 class，但是 Bean 还有其他几个属性。

| 属性 | 描述 |
| ------------- |:-------------:|
| class | 这个属性是强制性的，并且指定用来创建 bean 的 bean 类 |
| name | 这个属性指定唯一的 bean 标识符。在基于 XML 的配置元数据中，你可以使用 ID 或 name 属性来指定 bean 标识符（我一般用id） |
| scope | 这个属性指定由特定的 bean 定义创建的对象的作用域 |
| constructor-arg | 它是用来注入依赖关系的（构造器注入） |
| properties | 它是用来注入依赖关系的（setter注入） |
| autowiring mode | 它是用来注入依赖关系的（自动注入） |
| lazy-initialization mode | 延迟初始化的 bean 告诉 IoC 容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例 |
| initialization 方法 | 在 bean 的所有必需的属性被容器设置之后，调用回调方法 |
| destruction 方法 | 当包含该 bean 的容器被销毁时，使用回调方法 |

------------------------------
看表可能比较难懂，可以看一下伪代码：

```
<bean id="指定上下文唯一标识" class="类的全限定名称"
  lazy-init="懒加载（默认为false，懒加载的意思是等到对象创建时才进行加载，而不是在容器初始化时进行加载）"
   init-method="调用类中的一个初始化方法（只需要写方法名）"
    destroy-method="调用类中的一个销毁方法（只需要写方法名）"
     parent="父对象ID（这个相当于Java里面的继承关系）" >
	<!-- scope="默认为singleton（单例），我们可以手动修改为prototype" 添加范围后不能执行销毁方法 -->
	
		<property name="类中的属性或方法名" value="赋值"></property>
	</bean>
	
```

这大概就是 IOC 的基本概念和用法了，如写得不对，望指出，谢谢！
