---
layout: post
title: "Java数据类型转换"
date: 2017-2-13 19:42:53
categories: java
tags: [java, codeing]
---

Java中的类型转换操作

在java中，类型的转换是一种比较安全的操作，如果要执行一种名为榨化转换的操作(也就是说，将能容纳更多信息的数据类型转换成无法容纳那么多信息的类型)，
就有可能面临信息丢失的危险，所以此时，编译器会强制我们进行类型转换，所以说，这可能是意见危险的事情，如果无论如何都要这么做，那必须显式地进行类型转换．
而对于扩展转换，则不必显式地进行类型转换，因为新类型肯定能容纳原来类型的信息，不会造成任何信息的丢失

如下程序，java在将float或double类型转换成int类型时，总是对该数字执行截尾．

<!-- more -->

* 如果想要得到舍入的结构，就需要使用java.lang.Math中的round()方法

* 比int类型小的数据类型只有byte和short类型

* 通常，在表达式中出现的最大的数据类型决定了该表达式最终的数据类型，例如一个float值与一个double值相乘，结果就是double类型

* 如果一个int和一个long类型相加，则结果为long类型

* java没有sizeof,在c和c++中，sizeof()操作符可以告诉你为数据项分配的字节数，而通常使用sizeof()的最大原因是为了"移植"．

* 不同的数据类型在不同的机器上可能有不同的大小，所以在进行一些与存储空间有关的运算时，程序员必须获悉那些数据类型具体有多大．例如，
一台计算机可用32位来保存整数，而另一台只用16位来保存，显然，在第一台机器中，程序可保存更大的值，可以想想，移植是令c和c++程序员颇为头疼的一个问题，
而java则不需要sizeof()操作符来满足这方面的需要，所有数据类型在所有机器中的大小都是相同的

代码如下:
	
	public class CastingNum {
		public static void main(String [] args){
			double above = 0.7,below = 0.4;
			float fabove = 0.7f,fbelow = 0.4f;
			System.out.println("int(above)" + (int)above);
			System.out.println("int(below)" + (int)below);
			System.out.println("int(fabove)" + (int)fabove);
			System.out.println("int(fbelow)" + (int)fbelow);
			System.out.println("下面使用四舍五入的方法");
			System.out.println("Math.round(above)" + Math.round(above));
			System.out.println("Math.round(below)" + Math.round(below));
			System.out.println("Math.round(fabove)" + Math.round(fabove));
			System.out.println("Math.round(fbelow)" + Math.round(fbelow));
		}
	}

输出:

	int(above)0
	int(below)0
	int(fabove)0
	int(fbelow)0
	下面使用四舍五入的方法
	Math.round(above)1
	Math.round(below)0
	Math.round(fabove)1
	Math.round(fbelow)0


