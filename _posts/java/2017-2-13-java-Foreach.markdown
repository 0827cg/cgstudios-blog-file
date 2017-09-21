---
layout: post
title: "Java中的foreach语法"
date: 2017-2-13 19:49:13
categories: java
tags: [java, codeing]
---

foreach语法多用与数组和容器，表示不用创建int变量去对由访问项构成的序列进行计数，foreach将自动产生每一项

* 程序中Random random = new Random(47)中的47作为种子,是伪随机数生成器的内部状态的初始值，
随机产生的数字随着种子的改变而改变也就是使用了种子的随机数生成器其random对象在每一次运行程序时产生的数值都相同，

* 而Random random = new Random()这中创建对象可以认为是默认使用时间作为种子

代码如下:

<!-- more -->

	public class ForEachFloat {
		public static void main(String [] args){
			Random random = new Random(47);
			float f[] = new float[5];
			for(int i = 0; i < 5; i++){
				f[i] = random.nextFloat();
			}
			for(float x : f){
				System.out.print(x + "---");
			}
			System.out.println();
			TextVector textVector = new TextVector();
			textVector.showVector();
		}
	}
	class TextVector{
		Vector<Object> vector = new Vector<Object>();
		public void showVector(){
			for(int i = 1; i <= 5; i++){
				vector.add(i * 10);
			}
			for(Object x : vector){
				System.out.print(x + "---");
			}
		}
	}

输出:

	0.72711575---0.39982635---0.5309454---0.0534122---0.16020656---
	10---20---30---40---50---

