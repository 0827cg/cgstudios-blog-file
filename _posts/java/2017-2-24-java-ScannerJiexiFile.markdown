---
layout: post
title: "Java中Scanner类扫描文件"
date: 2017-2-23 19:35:52
categories: java
tags: [java, codeing]
---

Java中的Scanner类除了可以用来读取扫描从键盘输入,还可以扫描文件

Scanner类很基本,通过读取用户从键盘输入字符来实现交互.主要是通过Scanner类中的nextLine()方法来获取输入的所有内容,也可以使用next()方法,
但next()方法不会获取空格,但一般使用nextLine()方法

<!-- more -->

nextInt()则是获取键盘输入的数字.Scanner同IO输入输出流一样需要关闭,所以在程序退出时,需要调用close()方法将扫描器

例如代码:

	package com.cgtest.scannerDemo;

	import java.util.Scanner;

	public class ScannerDemo {
		public static void main(String [] args){
			Scanner scanner = new Scanner(System.in);
			System.out.println("请输入内容:");
			while(true){
				String inputStr = scanner.nextLine();
			
				if(inputStr.equals("exit")){
					scanner.close();
					break;
				}
				System.out.println(">>>" + inputStr);
			}
		}
	}

运行结果:

	请输入内容:
	憨头
	>>>憨头
	hey
	>>>hey
	666
	>>>666
	exit

可以用字符串,输入流(InputStream),文件等作为参数来创建Scanner对象.这里例子是用file用作参数来创建Scanner对象,就用创建之后的scanner对象来扫描此文件了.

扫描文件一般使用正则表达式来从文件中获取想要的内容

例如代码:

	// package com.cgstudy.s0922;

	import java.io.File;
	import java.io.FileNotFoundException;
	import java.util.Scanner;

	public class ScannerJiexi {                                      //使用正则表达式来解析文件
		public static void main(String [] args){
			File file = new File("cost.txt");
			Scanner sc = null;
			double sum = 0;
			try{
				sc = new Scanner(file);
				sc.useDelimiter("[^0123456789.]+"); //useDelimiter()将正则表达式作为分隔符
				while(sc.hasNextDouble()){
					double price = sc.nextDouble();
					sum = sum+price;
					System.out.println(price);
				}
				System.out.println("总共消费了"+sum+"元");
			}catch(FileNotFoundException e){
				e.printStackTrace();
			}
		}
	}

文件cost.txt中的内容是:

	TV cost 876 dollar,
	Computer cost 2398 dollar,
	The milk cost 98 dollar,
	The apples cost 198 dollar.

程序的输出为:

![java-bookContorlSystem](/images/java/java-ScannerJiexi.png)

Scanner对象能这样解析文件,更何况查看文件,它可以做到同IO中的输入流一样的作用,要达到这样的功能,只需更改一些代码即可,

如下代码片段:

	scanner = new Scanner(file);
	while(scanner.hasNextLine()){
		System.out.println(scanner.nextLine());
	}

只需要将这些的代码代替ScannerJiexi类中的try-catch中的代码块即可
之后程序的输出就为cost.txt的文本内容













