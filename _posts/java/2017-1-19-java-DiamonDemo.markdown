---
layout: post
title: "Java打印菱形  "
date: 2017-1-19 18:11:39
categories: java
tags: [java, codeing]
---

Java打印菱形和空心菱形，如图：

![java-diamon](/images/java/java-diamonDemo.png)

首先理一下思路，先看打印实心菱形，菱形可以看成是两个大小小正三角形，已中间的对角线为分割线来看，实例来说加入一个菱形总共有7行，那么可以看成是上部分是4行的正三角形，下部分是3行的正三角形，
那么也就是我们可以分开来打印。

<!-- more -->

先打印上部分，第一行3个空格，1个\*号，第二行2个空格，3个\*号，第三行1个空格，5个\*号，，先打印第一行，利用for循环打印，先打印空格，再打印\*号
ok,再找规律

|假设总共有4行 |第1行 |第2行|第3行|第4行|....|第n-1行|第n行|
|---|:---|:---|:---|:---|:---|:---|:---:|
|空格数|3|2|1|0|....|n-\(n-1\)|n-n|
|\*数量|1|3|5|7|....|\(\(n-1\)\*2\)-1|\(n\*2\)-1|

下部分则和上部分反过来看,道理是一样的
空心菱形则是则需要添加些if的判断语句，当在打印\*号时，当此时位置属于边界时，打印\*号，否则打印空格
代码如下:
	
	package com.cgtext.diamon;

	import java.util.Scanner;

	public class DiamonDemo {
		public static void main(String [] args){
			DrawShape drawShape = new DrawShape();
			while (true) {
				int c = drawShape.readChoice();
				if(c == 1){
					drawShape.drawShi(drawShape.readRows());
				}else if(c == 2){
					drawShape.drawKong(drawShape.readRows());
				}else if(c == 0){
					System.exit(0);
				}else{
					System.out.println("请输入可选序号0~2");
				}
			}
		}
	}
	
	class DrawShape{
		Scanner scanner = new Scanner(System.in);
		public int readChoice(){
			System.out.print("请输入所打印的形状的序号(1.打印实心菱形；2.打印空心菱形；0.退出)：");
			int c = scanner.nextInt();
			return c;
		}
		public int readRows(){
			System.out.println("请输入上部分行数：");
			int n = scanner.nextInt();
			return n;
		}
		public void drawShi(int n){
			for(int i = 1;i <= n;i++){                      //假设上部分有n行
				for(int x = n-i;x > 0;x--){              //打印空格数，随着行数的增加，空格数减少
						System.out.print(" ");
	//					sleep(300);      //调用后面定义的sleep(int n)方法,每运行一句输出语句休息300毫秒，只不过现在注销了
				}
				for(int y = 1;y <= (2*i)-1;y++){         //打印*号，随着行数增加，以(2*n)-1的规律增加
						System.out.print("*");
	//					sleep(300);
				}
				System.out.println();
			}
			for(int i = 1;i < n;i++){
				for(int x = 1;x<= n-(n-i);x++){           //第1行有一个空格，最后那行空格数为n-(n-i)
					System.out.print(" ");
	//				sleep(300);
				}
				for(int y = (n-i)*2-1;y > 0;y--){          //第1行有(n-i)*2-1个*号，最后一行有1个*号
					System.out.print("*");
	//				sleep(300);
				}
				System.out.println();
			}
		}
		public void drawKong(int n){
			for (int i = 1; i <= n; i++) {
				for(int x = n-i;x > 0;x--){
					System.out.print(" ");
	//				sleep(300);
				}
				for(int y = 1;y <= (2*i)-1;y++){
					if(y == 1 || y == (2*i)-1){
						System.out.print("*");
	//					sleep(300);
					}else{
						System.out.print(" ");
	//					sleep(300);
					}
				}
				System.out.println();
			}
			for(int i = 1;i < n;i++){
				for(int x = 1;x <= n-(n-i);x++){
					System.out.print(" ");
				}
				for(int y = (n-i)*2-1;y > 0;y--){
					if(y == ((n-i)*2-1) || y == 1){
						System.out.print("*");
	//					sleep(300);
					}else{
						System.out.print(" ");
	//					sleep(300);
					}
				}
				System.out.println();
			}
		}
		public void sleep(int n){
			try {
				Thread.sleep(n);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}

