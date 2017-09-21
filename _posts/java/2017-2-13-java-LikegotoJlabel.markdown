---
layout: post
title: "Java类似C中的goto跳转语句"
date: 2017-2-13 19:32:52
categories: java
tags: [java, codeing]
---

Java中的跳转标签机制label

##### 总结点:

* break:终端内部迭代，回到外部迭代

* continue：使执行点移到内部迭代的起始处

* continue label：同时中断内部迭代以及外部迭代，直接转到label处在执行，实际上是继续迭代过程，但却从外部迭代开始

<!-- more -->

* break label：中断所有迭代，并回到label处，但并不重新进入迭代，也就是说，是完全终止了两个迭代

* 所有的对比都是建立在for循环条件中i++之前的i值，而不是i++后的值

* continue label语句执行后，也会如同break语句一样，跳过for循环里的i++，

* 重点:在java里需要使用标签的唯一理由就是因为有循环嵌套存在，而且想从多层嵌套中break或continue

代码如下:

	public class LabelTest {
		public static void main(String [] args){
			int i = 0;
			outer:
				for(;;){
					inner:
						for(; i < 10; i++){
							System.out.println("开始i = " + i + "循环");
							if(i == 2){
								System.out.println("i == 2,执行continue,将执行i=３循环......");
								continue;
							}
							if(i == 3){
								System.out.println("i == 3,执行break,不执行后面的for(k)循环,直接进入for(i)的i=4循环......");
								i++;
								System.out.println("此时i的值为" + i);
								break;
							}
							if(i == 5){
								System.out.println("i == 5，执行continue outer，回到outer标签,从最外层for(;;)循环开始,将执行i=6循环......");
								i++;
								continue outer;
							}
							if(i == 9){
								System.out.println("i == 9，执行break outer,终止所有循环,程序结束");
								break outer;
							}
							for(int k = 0; k < 5; k++){
								if(k == 3){
									System.out.println("i = " + i + "的循环," + "k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = " + (i + 1) + "循环......");
									continue inner;
								}
							}
						}
				}
		}
	}

输出:

	开始i = 0循环
	i = 0的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 1循环......
	开始i = 1循环
	i = 1的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 2循环......
	开始i = 2循环
	i == 2,执行continue,将执行i=３循环......
	开始i = 3循环
	i == 3,执行break,不执行后面的for(k)循环,直接进入for(i)的i=4循环......
	此时i的值为4
	开始i = 4循环
	i = 4的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 5循环......
	开始i = 5循环
	i == 5，执行continue outer，回到outer标签,从最外层for(;;)循环开始,将执行i=6循环......
	开始i = 6循环
	i = 6的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 7循环......
	开始i = 7循环
	i = 7的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 8循环......
	开始i = 8循环
	i = 8的循环,k从０开始，k == 3－＞执行continue inner,回到inner处,从for(i)循环,将执行i = 9循环......
	开始i = 9循环
	i == 9，执行break outer,终止所有循环,程序结束



