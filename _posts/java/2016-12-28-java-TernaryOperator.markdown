---
layout: post
title: "Java三元操作符"
date: 2016-12-28 21:36:15
categories: java
tags: [java, codeing]
---

三元操作符也称为条件操作符,它显得比较独特,因为它有三个操作数;但它确实属于操作符的一种,因为它最终也会生成一个值.其表达式采取的形式如下
`boolean-exp ? value1 : value2`
如果boolean-exp\(布尔表达式\)的结果为true,则表达式所产生的值就为value1,如果boolean-exp的结果为false,值就计算为value2. 当然,如果需要达成这中需求这也可以用普通的if-else语句,但if-else语句相对于三元操作符来说,if-else显得相对复杂,三元操作符则更为简洁.

<!-- more -->

C语言中也有三元操作符,作为古老的编程语言,三元操作符就是C发明出来的,C引以为傲的就是它是一种简练的语言,而且三元操作符的引入多半就是为了体现这种高效的编程,但假如你打算频繁的使用它,那还得三思而后行,因为三元操作符使用多了很容易产生可读性极差的代码.

如下代码:

	public class TernaryOperatorTest {

		public static void main(String [] args){

			System.out.println(standardMethod(9));

			System.out.println(ternaryMethod(10));

			System.out.println(standardMethod(9));

		}

		static int ternaryMethod(int i){

			return i < 10 ? i * 100 : i * 10;

		}

		static int standardMethod(int i){

			if(i < 10){

				return i * 100;

			}else{

				return i * 10;

			}

		}
	}

Output:

900

100

900

通过上面的代码,可以看出,使用了三元操作符的方法ternarMethod\(\)显得更紧凑,
方法块更简练
而使用了普通if-else语句的方法standardMethod\(\)则更容易理解,而且不需要太多的录入.所以在选择使用三元操作符时,需要仔细考虑


