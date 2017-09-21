---
layout: post
title: "Java跳转语句"
date: 2017-2-13 19:13:27
categories: java
tags: [java, codeing]
---

Java中的break和continue语句

自己即将毕业了,准备把之前学习时编写的代码笔记整理下.重温知识点

* break用于强行退出循环，不执行循环中剩余的语句，而continue则是停止执行当前的迭代，然后退回循环起始处，开始下一次迭代

* foreach语句貌似不能像for循环那样依次存储数据，只能遍历输出数据

* 无穷循环有两种形式:while(true)和for(;;)

* 为了解break和continue的运行过程的差别,例如程序中的forbreakTest()方法

<!-- more -->

* 在当i == 2时，程序输出打印后，执行continue语句，在结束本次循环前,会先执行for循环中的条件语句i++，之后在结束本次循环

* 而break则不同，在当i == 7时，程序输出打印后，执行break语句，结束所有循环，for循环中的条件语句i++会未被执行，直接结束

代码如下:
	
	public class BreakContinue {
		public static void main(String [] args){
			Test test = new Test();
			test.forTest(50);
			test.foreachTest(40);
			test.forbreakTest();
		}
	}
	class Test{
		public void forTest(int maxNum){
			System.out.println("所查找出在数字0~" + maxNum + "中,能被３整除且不等于47的数有:");
			for(int i = 0, j = 1; i < maxNum; i++, j++){
				if(i == 47) break;
				if(i % 3 != 0) continue;
				System.out.println("第" + j + "次查找,i = " + i);
			}
		}
		public void foreachTest(int maxNum){
			int index[] = new int[10];
			Random random = new Random();
			for(int i = 0; i < index.length; i++){
				index[i] = random.nextInt(maxNum);
			}
			for(int i = 0; i < index.length; i++){
				System.out.println("index" + "[" + i + "]" + index[i]);
			}
			System.out.println("在随机生成元素０～" + maxNum + "的数组中不等于47且能被２整除的元素有:");
			for(int n: index){
				if(n == 47) break;
				if(n % 2 != 0) continue;
				System.out.print(n + "---");
			}
			System.out.println();
		}
		public void forbreakTest(){
			System.out.println("下面测试break和continue");
			for(int i = 0; i < 10; i++){
				System.out.print("i = " + i + ",");
				if(i == 2){
					System.out.println();
					System.out.println("i == 2,执行continue,结束本次循环,进入i=3循环");
					continue;
				}
				if(i == 7){
					System.out.println();
					System.out.println("i == 7,执行break,结束整个for循环");
					break;
				}
			}
		}
	}
 
输出:

	所查找出在数字0~50中,能被３整除且不等于47的数有:
	第1次查找,i = 0
	第4次查找,i = 3
	第7次查找,i = 6
	第10次查找,i = 9
	第13次查找,i = 12
	第16次查找,i = 15
	第19次查找,i = 18
	第22次查找,i = 21
	第25次查找,i = 24
	第28次查找,i = 27
	第31次查找,i = 30
	第34次查找,i = 33
	第37次查找,i = 36
	第40次查找,i = 39
	第43次查找,i = 42
	第46次查找,i = 45
	index[0]20
	index[1]10
	index[2]16
	index[3]0
	index[4]37
	index[5]4
	index[6]19
	index[7]35
	index[8]13
	index[9]15
	在随机生成元素０～40的数组中不等于47且能被２整除的元素有:
	20---10---16---0---4---
	下面测试break和continue
	i = 0,i = 1,i = 2,
	i == 2,执行continue,结束本次循环,进入i=3循环
	i = 3,i = 4,i = 5,i = 6,i = 7,
	i == 7,执行break,结束整个for循环



