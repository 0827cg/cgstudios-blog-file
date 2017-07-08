---
layout: post
title: "JavaEE-Spring的helloworld"
date: 2017-1-3 11:15:27
categories: javaEE
tags: [javaEE, codeing]
---
使用Spring框架,编写一个HelloWorld小程序.要求可选择性输出中文\"大家好\"和英文\"Hello EveryOne\",程序控制流程图如下图所示:

![spring-helloworld](/images/javaee/spring-helloworld.png)

这里用文字描述下程序流程图的大概过程.利用一个接口,定义一个具有返回值的抽象方法,其接口有两个实现类HelloWorld和HelloChina,分别用来返回\"Hello Everybody\"和\"大家好\"信息,并通过Person类来接受信息.在程序的main方法中,先读取xml配置文件,通过xml配置文件中代码来控制程序具体由那个实现类来返回信息,之后再调用Person类来进行输出所返回的信息.

<!-- more -->

在Eclipse中创建工程之后,需要先搭建Spring框架.在[官网上下载Spring压缩包][].在工程中新建lib文件夹,将解压后的libs文件夹里的文件全部复制到工程的lib文件夹下\(原本只需要4个jar文件:Spring.Core.jar,Spring.Beans.jar,Spring.AOP.jar和Spring.Context.jar\),再将其都添加到Build Path路径中.

除此之外,将Spring工具包全添加进去之后,还需要另外下载[commons-logging-1.2-bin.zip][],解压后将其中的commons-logging-1.2.jar文件同样复制到工程的lib文件夹下,并添加到Build Path路径中.至此,环境才算搭建完成.即可开始创建Java文件了

在Eclipse中创建实例工程.在这里,我们在工程中逐个创建Java文件和xml配置文件:

* ShowhelloMessage:一个接口,用于定义输出问候信息
* HelloWorld:接口的实现类,返回"Hello Everybody"信息
* HelloChina:接口的实现类,返回"大家好"信息
* Person:一个人物类,调用ShowhelloMessage接口,接受信息后向用户打印问候信息
* spring-HelloWorld.xml:xml配置文件
* SpringHelloWorld:程序的入口类,用户加载配置文件并启动IOC容器,调用人物类,向用户打印问候信息

也就是说,需要创建5个Java文件,其通过配置文件来选择执行.降低了各个文件之间的耦合度,这就是使用Spring的好处

如下代码:

<font color="#242424">ShowhelloMessage:</font>

	package com.springhelloworld.action;
	
	public interface ShowhelloMessage {
	
		public String showMessage();
	
	}
	
<font color="#242424">HelloWorld:</font>

	package com.springhelloworld.action;
	
	public class HelloWorld implements ShowhelloMessage {
	
		@Override
		public String showMessage() {
			return "Hello Everybody";
		}
	}
	
<font color="#242424">HelloChina:</font>

	package com.springhelloworld.action;
	
	public class HelloChina implements ShowhelloMessage{
	
		@Override
		public String showMessage() {
			return "大家好";
		}
	}

<font color="#242424">Person:</font>

	package com.springhelloworld.action;
	
	public class Person {
	
		private ShowhelloMessage showhelloMessage;
		
		public void setshowhelloMessage(ShowhelloMessage showhelloMessage){
			this.showhelloMessage = showhelloMessage;
		}
	
		public ShowhelloMessage getshowhelloMessage(){
			return showhelloMessage;
		}
		public String sayHello(){
			return this.showhelloMessage.showMessage();
		}
	}
	
<font color="#242424">spring-HelloWorld.xml:</font>

	<?xml version = "1.0" encoding = "UTF-8"?>
	<!DOCTYPE beans PUBLIC "-//SPRING/DTD BEAN/EN" "http://	www.springframework.org/dtd/spring-beans.dtd">
	<beans>
		<bean id = "helloWorld" class = "com.springhelloworld.action.HelloWorld"></bean>
		<bean id = "helloChina" class = "com.springhelloworld.action.HelloChina"></bean>
		<bean id = "person" class = "com.springhelloworld.action.Person">
			<property name="showhelloMessage" ref="helloWorld"></property>
		</bean>
	</beans>
	
<font color="#242424">SpringHelloWorld:</font>

	package com.springhelloworld.main;
	
	import org.springframework.beans.factory.BeanFactory;
	import org.springframework.beans.factory.xml.XmlBeanFactory;
	import org.springframework.core.io.FileSystemResource;
	import org.springframework.core.io.Resource;
	
	import com.springhelloworld.action.Person;
	
	public class SpringHelloWorld {
	
		public static void main(String [] args){
		
			Resource resource = new FileSystemResource("spring-HelloWorld.xml");
			BeanFactory beanFactory = new XmlBeanFactory(resource);
			Person person = (Person)beanFactory.getBean("person");
			String str = person.sayHello();
			
			System.out.println("The person is currently saying " + str);
		}
	}
	
Output:

<font color="f08080">十二月 15, 2016 7:04:30 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from file [/home/cg/Work/JavaWork/Eclipse-161103-restudy/Spring-HelloWorld/spring-HelloWorld.xml]</font>
The person is currently saying Hello Everybody

*

认真阅读上面的代码,可以看出此程序使用的依赖注入的方式是Setter注入方式.在Person类中为使用接口类ShowhelloMessage而编写了set和get方法.并在Person类中定义sayHello\(\)方法来获取抽象类的实现类所返回的问候信息.

其次,在xml配置文件中:

 `<beans>`标签类似html标签中的`<body>`标签,而`<bean>`是被包含在`<beans>`里面的子标签.每一个`<bean>`标签代表一个类,其中的id属性值用来区分各个类,并在class属性中指明类所在的包的路径.而`<property>`标签是为描述自己所在`<bean>`标签的属性值,name的属性值为Person类中声明的私有属性

ShowhelloMessage的对象showhelloMessage,ref属性则表示制定哪个类来返回问候信息,其值就是之前定义的`<bean>`标签的id值.上面代码中的`ref = "helloWorld"`则表示配置文件制定调用HelloWorld这个实现类来返回问候信息,所以程序的输出就为英文\"Hello Everybody\",所以如果需要让程序输出\"大家好\",则只需要修改ref的属性值改为\"helloChina\"即可

在程序的main方法中,代码:

`Resource resource = new FileSystemResource("spring-HelloWorld.xml")`

表示利用FileSystemResource来加载读取配置文件,并返回一个Resource类型的对象.其FileSystemResource这个类加载配置文件的路径是从工程的根目录开始,而这里的spring-HelloWorld.xml文件就存放在工程的根目录下,所以名字就是路径名

代码`BeanFactory beanFactory = new XmlBeanFactory(resource)`则是利用XmlBeanFactory来加载配置文件从而启动IOC容器,其参数就是FileSystemResource在加载配置文件时所返回的Resource类型的对象.此代码执行后将得到一个BeanFactory类型的对象,利用此对象使用BeanFactory类中的getBean\(String beanID\)方法从IOC容器中获取所需要的bean所指向的类的对象,就如同下一行代码:

`Person person = (Person)beanFactory.getBean("person")` 

指从IOC容器中获取Person类的实例,这里使用了类型的强制转换

[官网上下载Spring压缩包]:http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#overview-distribution-zip
[commons-logging-1.2-bin.zip]:http://commons.apache.org/proper/commons-logging/download_logging.cgi
