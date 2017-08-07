---
layout: post
title: "JavaEE-Hibernate框架概况"
date: 2017-3-08 19:48:02
categories: javaEE
tags: [javaEE, codeing]
---

复习Hibernaet框架.知识重温

### Hibernate框架简述

总得来说,它就是用来操作数据库的.在JavaEE开发方面来说,它这个框架所用在层面是数据访问层,在系统上来说算是底层的操作
Hibernate框架其主要是封装了JDBC的一系列操作.使用Hibernate框架可以使得系统在于数据库进行数据传递时不需要用到SQL语句,其实现的方法是实体类于数据库表之间的映射关系,也即是ORM(Object Relation Mapping)对象关系映射
通过ORM,使得系统在获取数据时可以直接从实体类中获取,而不需要一味的去操作数据库.ORM映射机制就相当于在实体类和数据库表中搭建了一座桥梁,数据库表中的数据通过桥梁,将数据赋值给实体类中的属性,根据面向对象系统只需要获取此时的实体类的对象就可以获取到其数据,并进行对数据的操作.反过来,数据在被系统操作之后,新的数据就被重新赋值给了实体类的属性,系统再通过Hibernate的ORM机制,将此时实体类的对象里的属性存储到数据库表中,完成数据的存储.

<!-- more -->

当中的读取和存储过程都不需要用到sql语句来对数据库表进行操作,这样使用Hibernate就简化了系统对数据库的访问操作.所以,Hibernate框架就可以简单的理解为是为使系统能简单的只从实体类中就可以获得数据的一种工具,为达到上面的目的,想想Hibernate要怎么做才能实现仅通过ORM而不是用sql语句就可以从数据库中获取数据呢
这就是它的奥妙之处.
Hibernate通过在实体类和数据库表中间建立一种映射机制,实现映射机制就需要用到映射文件,就是实体类和对应的数据库表之间的映射文件
参照例子代码:

* 实体类Student
* 实体类映射配置文件Student.hbm.xml
* Hibernate核心配置文件hibernate.cfg.xml

#### Student.java

有一个实体类Student,代码如下:

	package com.cgtest.beans;

	public class Student {
	
		private String stuId;
		private String stuName;
		private String stuSex;
		private int stuAge;
		private String stuAddress;
		private String stuTel;
	
		public String getStuId() {
			return stuId;
		}
		public void setStuId(String stuId) {
			this.stuId = stuId;
		}
		public String getStuName() {
			return stuName;
		}
		public void setStuName(String stuName) {
			this.stuName = stuName;
		}
		public String getStuSex() {
			return stuSex;
		}
		public void setStuSex(String stuSex) {
			this.stuSex = stuSex;
		}
		public int getStuAge() {
			return stuAge;
		}
		public void setStuAge(int stuAge) {
			this.stuAge = stuAge;
		}
		public String getStuAddress() {
			return stuAddress;
		}
		public void setStuAddress(String stuAddress) {
			this.stuAddress = stuAddress;
		}
		public String getStuTel() {
			return stuTel;
		}
		public void setStuTel(String stuTel) {
			this.stuTel = stuTel;
		}
	}

实体类中的属性都是private的,属性就是用来临时存放相对应的数据的,这些数据就是从与实体类相对应相映射的数据库表中得来的,那么就必须会有一张表存放这个实体类对应的数据,就比如这个表中的数据都有6个字段与上面的实体类中的6个属性值相对应,那么再需要一个映射文件来将表和类连接起来
那么对应上面

#### Student.hbm.xml

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE hibernate-mapping PUBLIC
		"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
		"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
	
	<hibernate-mapping>
		<!-- 配置类和表对应
				class标签:	
					name属性值:实体类全路径
					table属性值:数据库中表的名字
		 -->
		<class name="com.cgstudy.student.Student" table="dt_t_stuMsg">
			<!-- 配置实体类id和表id对应 
				hibernate要求实体类有一个属性是唯一值,也就是值不能重复
				同样也要求表中的字段有唯一值
			-->
			<!-- id标签
				name属性值:实体类里id(唯一值)的属性名称(这里的是学号为主键,学号是唯一的)
				column属性值:生成表的字段名称,一般跟name的值一样
			 -->
			<id name="stuId" column="stuId">
				<!-- 设置数据库表id增长策略 
						native:使生成的id值为主键并自动增长
				-->
				<generator class="assigned"></generator>
			</id>
			<!-- 配置实体类中其他属性和表中的字段相对应-->
			<!-- property标签
					name属性值:实体类的属性值
					column属性值:表中的字段的名字,一般跟name值一样
			 -->
			 <property name="stuName" column="stuName"></property>
			 <property name="stuSex" column="stuSex"></property>
			 <property name="stuAge" column="stuAge"></property>
			 <property name="stuAddress" column="stuAddress"></property>
			 <property name="stuTel" column="stuTel"></property>
		</class>
	</hibernate-mapping>

代码块中已经有了注释,就不再解释代码的意思了,总之,需要有专门的映射文件将实体类和数据库表相连接起来.另外,Hibernate会自动在数据库中创建表,只是表,不会创建数据库.其创建表的依据就是根据实体类的属性以及映射文件的相关配置来创建表.例如上面映射文件中的class标签中的table属性值为"dt_t_stuMsg",那么在系统运行后,将会在数据库中创建一个名为dt_t_stuMsg的表,这个表就是跟Student.java这个实体类相互映射的
这里总结一下知识点,顺便用来承上启下

* 实体类和表的映射文件名格式:xxx.hbm.xml(xxx一般为实体类的名字)
* 映射文件的存放位置一般在实体类所在的包下
* Hibernate核心配置文件的命名格式为:hibernate.cfg.xml(固定的)
* Hibernate核心配置文件存放位置在src文件夹下(固定的)
* 解压后其中的lib目录下的就是我们开发所需要的jar包，其中required是我们所必须需要的包,同样需要数据库驱动jar包

#### hibernate.cfg.xml

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE hibernate-configuration PUBLIC
		"-//Hibernate/Hibernate Configuration DTD 3.0//EN"
		"http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
	<hibernate-configuration>
		<session-factory>
			<!-- 配置数据库信息,必须要有 -->
			<property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
			<property name="hibernate.connection.url">jdbc:mysql://localhost:3306/dt_stuMsg</property>
			<property name="hibernate.connection.username">root</property>
			<property name="hibernate.connection.password">0827</property>
		
			<!-- 配置Hibernate信息,可选 -->
			<!-- 输出底层sql语句 -->
			<property name="hibernate.show_sql">true</property>
			<!-- 格式化sql语句,使语句更整洁 -->
			<property name="hibernate.format_sql">true</property>
			<!-- Hibernate帮创建表,需要配置之后 
				update:如已有表,则更新,若无,则创建
			-->
		
			<property name="hibernate.hbm2ddl.auto">update</property>
		
		
			<!-- 配置数据库方言 -->
			<property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
		
			<!-- 把映射文件放到这里核心配置文件中,必须 
				resource的属性值为相对路径
			-->
			<mapping resource="/com/cgstudy/student/Student.hbm.xml"/>
		
		</session-factory>
	</hibernate-configuration>


### Hibernate核心配置文件的作用

* 连接数据库
* 加载映射文件
* 配置Hibernate

### 使用Hibernate框架用到的一些代码和对象

#### 代码

	Configuration configuration = null;
	SessionFactory sessionFactory = null;
	Session session = null;
	Transcation transcation = null;
	try{
		//加载核心配置文件
		configuration =  new Configuration().configure();
		//创建sessionFactory对象,并读取核心配置文件内容
		//在过程中会根据硬映射关系,在数据库中创建表
		sessionFactory = configuration.buildSessionFactory();
		//创建session对象.类似连接
		session = sessionFactory.openSession();
		//开启事务
		transaction = session.beginTransaction();
		.....
		.....//操作数据
		.....
		//事务的提交
		transaction.commit();
	}catch(Exception e){
		//事务的回滚
		transaction.rollback();
	}finall{
		//关闭事务
		transcation.close();
		//关闭session
		session.close();
	}

其中,在创建SessionFactory对象的过程中,将会特别消耗资源

#### SessionFactory类

这里了解下SessionFactory这个类,它是一个接口,一个SessionFactory实例对应一个数据存储源,应用从SessionFactory中获取Session实例.
SessionFactory特点:

* 线程是安全的,这意味着它的同一个实例可以被应用的多个线程共享
* 它是重量级的,意味着不能随意创建或销毁它的实例
* 如果应用只访问一个数据库,则只需要创建一个SessionFactory实例,在应用初始化时创建该实例
* 如果同时访问多个数据库,则需要对每个数据库单独创建一个SessionFactory实例

所以,考虑资源损耗方面,一般一个项目中就只创建一个SessionFactory对象,所以这里的解决方法是为SessionFactory专门创建一个工具类,放在系统中,给系统其他需要用到的SessionFactory使用
如代码:

	public class HibernateUtils{
		private static Configuration configuration = null;
		private static SessionFactory sessionFactory = null;
	
		static{
			configuration = new Configuration().configure();
			sessionFactory = configuration.buildSessionFactory();
		}
		public static SessionFactory getSessionFactory(){
			return sessionFactory;
		}
	}

这样的话,得到的sessionFactory对象就不用关闭了

#### Session类

其中的Session类也是一个接口
Session接口是Hibernate应用里使用最广泛的接口
Session也被称为持久化管理器,它提供了和持久化相关的操作,如添加,更新,删除,加载和查询对象
特点
不是线程安全的,因此在设计软件架构时,应该避免多个线程共享一个Session实例
轻量级的.它的创建和销毁不需要消耗太多的资源
Session类似jdbc中的connection
Session对象是单线程对象,不能共用
session操作数据常用的方法
* 添加:save()方法
* 修改:update()方法
* 删除:delete()方法
* 根据主键查询:get()方法,load()方法

在而,代码
`transcation.rollback()`
实现了事务的回滚,那么也就是说Hibernate类似与git命令一样,有缓存机制,即Hibernate的一级缓存.它的缓存就是把数据存放到内存中,省去了Hibernate重复读取数据库数据的步骤,但系统出现异常,可以将内存中的原始的缓存数据重新存放到数据库中,避免了数据的丢失

### Hibernate里可以实现查询的对象

#### Query对象

需要用到hql(Hibernate Query Language)语句,它跟sql相似
hql和sql的区别

* sql操作的是表和表字段
* hql操作的是实体类和属性

查询所有数据的hql语句:from 实体类名称
例

	Query query = session.createQuery("from Student");
	List<Student> list = query.list();
	for(Student student : list){
		System.out.println(student);
	}

#### Criteria对象
其不需要用到任何语句，查询所有数据的代码

	Criteria criteria = session.createCriteria(Student.class);
	List<Student> list = criteria.list();
	for(Student student : list){
			System.out.println(student);
	}

#### SQLQuery对象
可以使用SQL语句来操作

	SQLQuery sqlQuery = session.createSQLQuery("SELECT * FROM td_t_stuMsg");
	List<Object[]> list = sqlQuery.list();
	for(Object[] object : list){
		System.out.println(Arrays.toString(object));
	}
	
其中，这里获取到的list集合是二维的集合,如果需要得到和上面两个Query和Criteria对象获取的Student的所有对象信息,那么
代码如下

	SQLQuery sqlQuery = session.createSQLQuery("SELECT * FROM td_t_stuMsg");
	sqlQuery.addEntity(Student.class);
	List<Student> list = sqlQuery.list();
	for(Student student : list){
		System.out.println(student);
	}

Query,Criteria,SQLQuery这三个对象,通常会使用前两个,第三个使用的比较少,Hibernate的基本内容先写到这,当然还有许多内容并未总结,例如表的关系的配置等.到后面再慢慢总结

谢谢
