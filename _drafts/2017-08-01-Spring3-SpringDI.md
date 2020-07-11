---
layout: post
title: Spring-（3）Spring DI
categories: [Java, Spring]
description: Spring DI
keywords: Java, Spring
---

# Spring-（3）Spring DI

**DI：Dependency Injection，即「依赖注入」，依赖注入可以有高效的解耦合的能力。可以说，依赖注入仅仅是控制反转的一个具体的例子，当编写一个复杂的 Java 应用程序时，应用程序类应该尽可能的独立于其他的 Java 类来增加这些类可重用的可能性，当进行单元测试时，可以使它们独立于其他类进行测试。依赖注入有助于将这些类粘合在一起，并且在同一时间让它们保持独立。我们将这两个部分拆开来理解，依赖的关系就像是类里面套着另一个类，例如车店是一个类，车店类里面有一个属性是车主类，那么车店类就依赖了车主类，这就是依赖关系，那注入的意思是车主类通过IOC容器注入到车店！**

每个基于应用程序的 java 都有几个对象，这些对象一起工作来呈现出终端用户所看到的工作的应用程序。当编写一个复杂的 Java 应用程序时，应用程序类应该尽可能独立于其他 Java 类来增加这些类重用的可能性，并且在做单元测试时，测试独立于其他类的独立性。依赖注入（DI）有助于把这些类粘合在一起，同时保持他们独立。

Spring 有两种依赖注入类型：

- Constructor-based dependency injection：当容器调用带有多个参数的构造函数类时，实现基于构造函数的 DI，每个代表在其他类中的一个依赖关系;
- Setter-based dependency injection：基于 setter 方法的 DI 是通过在调用无参数的构造函数或无参数的静态工厂方法实例化 bean 之后容器调用 beans 的 setter 方法来实现的。

你可以混合这两种方法，基于构造函数和基于 setter 方法的 DI，然而使用有强制性依存关系的构造函数和有可选依赖关系的 setter 是一个好的做法。

现在我们从一个例子里面了解一下依赖注入的实现，我们创建两个 POJO 类，分别为 Company 类和 Person 类。

Company 类：

```
public class Company {
	private Person person;
	
	public Company() {}

	public Company(Person person) {
		System.out.println("公司有很多人！");
		this.person = person;
	}

	public void setPerson(Person person) {
		System.out.println("公司有很多人！");
		this.person = person;
	}
}
```

Person 类：

```
public class Person {
	private String say;
	
	public Person() {}
	
	public Person(String say) {
		System.out.println(say);
		this.say = say;
	}
	
	public void setSay(String say) {
		System.out.println(say);
		this.say = say;
	}
}
```

那么我们开始分别对基于构造函数和基于 setter 进行依赖注入，在 Beans.xml 文件中添加下面相应的代码块，这里我们先使用 xml 进行配置（以后讲到注解再用注解进行配置）

----------------------------------------------------------------
## 基于构造函数的依赖注入

```
<bean id="constructorCompany" class="com.tbspringDI.constructor.Company" lazy-init="true">
		<constructor-arg ref="constructorPerson"></constructor-arg>
	</bean>
	
	<bean id="constructorPerson" class="com.tbspringDI.constructor.Person" lazy-init="true">
		<constructor-arg type="java.lang.String" value="我是人，我说了一句人话！"></constructor-arg>
	</bean>
```

----------------------------------------------------------------
## 基于setter的依赖注入

```
<bean id="setterCompany" class="com.tbspringDI.setter.Company" lazy-init="true">
		<property name="person" ref="setterPerson"></property>
	</bean>
	
	<bean id="setterPerson" class="com.tbspringDI.setter.Person" lazy-init="true">
		<property name="say" value="我是人，我说了一句人话！"></property>
	</bean>
```

需要注意的是：基于构造函数的依赖注入需要 POJO 存在相应的构造方法，而基于 setter 的依赖注入则需要 POJO 存在相应的 setXxx() 方法。

下面用main方法进行一下测试：

```
public class MainAPP {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
		//基于构造函数注入就将下面getBean的（）里面改为constructorCompany
		Company company = (Company) context.getBean("setterCompany");
	}
}
```

控制台输出下面两句话我们就成功啦！

![](..\images\posts\java\Spring-3-Print1.png)

但为什么我们拿得是Company对象会输出两句话呢？

这是因为当我们从 IOC 容器中取 Company 时，IOC 帮我们先把 Person 初始化了一遍，所以我们会看到控制台会先输出一句「我是人，我说了一句话！」


---------------------
除了以上两种用法外，DI 还有另外的两种两种用法：

（一）innerBean（内部 Bean）：在 Beans.xml 中替换相应 Bean 即可。

```
<bean id="outerBeansCompany" class="com.tbspringDI.innerBeans.Company" lazy-init="true">
		<property name="person">
			<bean id="innerBeansPerson" class="com.tbspringDI.innerBeans.Person">
				<property name="say" value="我是人，我说了一句话！"></property>
			</bean>
		</property>
</bean>
```

（二）集合注入：看个用例，我们创建一个 JavaCollection 类并在 Beans.xml 中进行配置。

JavaCollection 类：

```
public class JavaCollection {
	private List addressList;
	private Set  addressSet;
	private Map  addressMap;
	private Properties addressProp;
	
	public List getAddressList() {
		System.out.println("List Elements :"  + addressList);
		return addressList;
	}
	public void setAddressList(List addressList) {
		this.addressList = addressList;
	}
	public Set getAddressSet() {
		System.out.println("Set Elements :"  + addressSet);
		return addressSet;
	}
	public void setAddressSet(Set addressSet) {
		this.addressSet = addressSet;
	}
	public Map getAddressMap() {
		System.out.println("Map Elements :"  + addressMap);
		return addressMap;
	}
	public void setAddressMap(Map addressMap) {
		this.addressMap = addressMap;
	}
	public Properties getAddressProp() {
		System.out.println("Property Elements :"  + addressProp);
		return addressProp;
	}
	public void setAddressProp(Properties addressProp) {
		this.addressProp = addressProp;
	}
	
}
```

Beans.xml 中添加下列代码块：

```
<!-- 注入集合 -->
<bean id="javaCollection" class="com.tbspringDI.setCollection.JavaCollection">

      <!-- 注入一列值，允许重复 -->
      <property name="addressList">
         <list>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
         </list>
      </property>

      <!-- 注入一列值，不允许重复 -->
      <property name="addressSet">
         <set>
            <value>INDIA</value>
            <value>Pakistan</value>
            <value>USA</value>
            <value>USA</value>
        </set>
      </property>

      <!-- 键值对，可以是任意类型 -->
      <property name="addressMap">
         <map>
            <entry key="1" value="INDIA"/>
            <entry key="2" value="Pakistan"/>
            <entry key="3" value="USA"/>
            <entry key="4" value="USA"/>
         </map>
      </property>

      <!-- 键值对，String类型 -->
      <property name="addressProp">
         <props>
            <prop key="one">INDIA</prop>
            <prop key="two">Pakistan</prop>
            <prop key="three">USA</prop>
            <prop key="four">USA</prop>
         </props>
      </property>

   </bean>
```

然后我们写个测试类（MainAPP）进行一下测试：

```
public class MainAPP {
	   public static void main(String[] args) {
	      ApplicationContext context = new ClassPathXmlApplicationContext("Beans.xml");
	      JavaCollection jc=(JavaCollection)context.getBean("javaCollection");
	      jc.getAddressList();
	      jc.getAddressSet();
	      jc.getAddressMap();
	      jc.getAddressProp();
	   }
}
```

输出结果如下：

![](..\images\posts\java\Spring-3-Print2.png)

我们可以看到以下区别：
- list 的列值是可以重复的；
- set 的列值是不能重复的；
- map 输出的是 Object 类型的键值对；
- Properties 输出的是 String 类型的键值对。

以上就是 Spring 依赖注入的用法，仅供学习参考。
