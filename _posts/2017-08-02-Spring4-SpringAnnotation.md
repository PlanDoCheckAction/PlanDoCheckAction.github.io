---
layout: post
title: Spring-（4）Spring 注解
categories: [Java, Spring]
description: Spring 注解
keywords: Java, Spring
---

# Spring-（4）Spring 注解

Spring 基于注解的配置：

从 Spring 2.5 开始就可以使用注解来配置依赖注入。而不是采用 XML 来描述一个 bean 连线，你可以使用相关类，方法或字段声明的注解，将 bean 配置移动到组件类本身。

如果需要用到注解进行依赖注入，则需要在 xml 文档中的 beans 中加上这一代码：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context-4.0.xsd">
	<!-- 启用注解进行依赖注入 -->
   <context:annotation-config/>

</beans>
```

配置完后，你就可以开始注解你的代码，表明 Spring 应该自动连接值到属性，方法和构造函数。

下面是几个重要的注解：

| 注解 | 说明 |
| ------------- |:-------------:|
| @Required | @Required 注解应用于 bean 属性的 setter 方法。 |
| @Autowired | @Autowired 注解可以应用到 bean 属性的 setter 方法，非 setter 方法，构造函数和属性。 |
| @Qualifier | 通过指定确切的将被连线的 bean，@Autowired 和 @Qualifier 注解可以用来删除混乱。 |
| JSR-250 Annotations | Spring 支持 JSR-250 的基础的注解，其中包括了 @Resource，@PostConstruct 和 @PreDestroy 注解。 |


-------------------------------------------------

各来一个例子↓直接贴代码

## @Required

xml：

```
<!-- 使用注解方式进行方法上的自动装配  @Required -->
   <bean id="companyRequired" class="com.tbspringAnnotation.required.Company">
   	<!-- <constructor-arg index="1" value="某公司"></constructor-arg> --><!-- 构造函数赋值 -->
   	<property name="companyName" value="某公司"></property><!-- setter赋值 -->
   	<property name="person" ref="personRequired"></property>
   </bean>
   
   <bean id="personRequired" class="com.tbspringAnnotation.required.Person">
   	<property name="say" value="我是人，我可以说话 ！"></property>
   </bean>
```

Company：

```
public class Company {
	
	private Person person;
	private String companyName;

	public String getCompanyName() {
		return companyName;
	}

	@Required  //方法上的自动装配  @Required只能是setter上的注解并且这个参数的bean必须存在，否则报错
	public void setCompanyName(String companyName) {
		this.companyName = companyName;
	}

	public Person getPerson() {
		return person;
	}

	@Required  
	public void setPerson(Person person) {
		System.out.println("公司有很多人！");
		this.person = person;
	}
}
```

Person：

```
public class Person {
	private String say;

	public void setSay(String say) {
		System.out.println(say);
		this.say = say;
	}
	
}
```

Main 测试：

```
public class MainAPP {
	   public static void main(String[] args) {
	      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
	      Company company =(Company)context.getBean("companyRequired");
	   }
}
```

-------------------------------------------
## @Autowired

xml：

```
<!-- 使用注解方式进行方法上的/属性上的/构造函数上的自动装配  @Autowired -->
   <bean id="companyAutowired" class="com.tbspringAnnotation.autowired.Company">
   	<!-- <constructor-arg index="1" value="某公司"></constructor-arg>--><!--构造函数赋值 -->
   	<property name="companyName" value="某公司"></property><!-- setter赋值 -->
   </bean>
   
   <bean id="personAutowired" class="com.tbspringAnnotation.autowired.Person">
   	<property name="say" value="我是人，我可以说话 ！"></property>
   </bean>
```

Company：

```
public class Company {
	//@Autowired  //属性上的自动装配
	private Person person;
	private String companyName;

	//@Autowired  //构造函数上的自动装配
	/*public Company(Person person, String companyName) {
		System.out.println("Company ：" + companyName);
		this.person = person;
		this.companyName = companyName;
	}*/

	public String getCompanyName() {
		return companyName;
	}

	@Autowired(required=false)  //required=false表示这个自动装配不是必须的
	public void setCompanyName(String companyName) {
		this.companyName = companyName;
	}

	public Person getPerson() {
		return person;
	}

	@Autowired  //方法上的自动装配
	public void setPerson(Person person) {
		System.out.println("公司有很多人！");
		this.person = person;
	}
}
```

Person：

```
public class Person {
	private String say;

	public void setSay(String say) {
		System.out.println(say);
		this.say = say;
	}
	
}
```

Main 测试：

```
public class MainAPP {
	   public static void main(String[] args) {
	      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
	      Company company =(Company)context.getBean("companyAutowired");
	   }
}
```

-------------------------------------
## @Qualifier

xml：

```
<!-- 使用注解方式进行方法上的/属性上的/构造函数上的自动装配  @Qualifier -->
    <bean id="companyQualifier" class="com.tbspringAnnotation.qualifier.Company">
   	<property name="companyName" value="某公司"></property>
   </bean>
   
   <bean id="personQualifier1" class="com.tbspringAnnotation.qualifier.Person" lazy-init="true">
   	<property name="say" value="我是人，我可以说话 ！"></property>
   </bean> 
   
   <bean id="personQualifier2" class="com.tbspringAnnotation.qualifier.Person" lazy-init="true">
   	<property name="say" value="哈哈哈哈哈哈哈哈哈 ！"></property>
   </bean>
```

Company：

```
public class Company {
	
	private Person person;
	private String companyName;


	public String getCompanyName() {
		return companyName;
	}

	public void setCompanyName(String companyName) {
		this.companyName = companyName;
	}

	public Person getPerson() {
		return person;
	}

	@Autowired
	@Qualifier("personQualifier2")//指定bean中对应的名称
	public void setPerson(Person person) {
		System.out.println("公司有很多人！");
		this.person = person;
	}
}
```

Person：

```
public class Person {
	private String say;

	public void setSay(String say) {
		System.out.println(say);
		this.say = say;
	}
}
```

Main 测试：

```
public class MainAPP {
	   public static void main(String[] args) {
	      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
	      Company company =(Company)context.getBean("companyQualifier");
	   }
}
```

--------------------------
最后，控制台输出：

![](..\images\posts\java\Spring-4-Print.png)

