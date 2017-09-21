---
layout: post
title: "Java笔试"
date: 2017-3-16 16:12:40
categories: java
tags: [java, codeing, 笔试题]
---

昨天邮件收到一份笔试题目,全英文.
答题时间是一个小时,本来信心满满,但自己翻译弄懂题目就花了半个多小时,哎 英语是硬伤.

### ok,上题目:

题目（共两题，考核基础代码能力而不是算法好坏）：

Problem Statement # 1

Given a set of N \(> 1\) positive integers, you are supposed to partition them into two disjoint sets A1 and A2 of n1 and n2 numbers, respectively. Let S1 and S2 denote the sums of all the numbers in A1 and A2, respectively. You are supposed to make the partition so that \|n1 - n2\| is minimized first, and then \|S1 - S2\| is maximized.

<!-- more -->

Input Specification:
Each input file contains one test case. For each case, the first line gives an integer N (2 <= N <= 10^5), and then N positive integers follow in the next line, separated by spaces. It is guaranteed that all the integers and their sum are less than 2^31.

Output Specification:
For each case, print in a line two numbers: \|n1 - n2\| and \|S1 - S2\|, separated by exactly one space.

 
Sample Input 1:

10

23 8 10 99 46 2333 46 1 666 555

Sample Output 1:

0 3611


Sample Input 2:

13

110 79 218 69 3721 100 29 135 2 6 13 5188 85

Sample Output 2:

1 9359

------

Problem Statement # 2

You are to write a program that takes a list of strings containing integers and words and returns a sorted version of the list.
 
The goal is to sort this list in such a way that all words are in alphabetical order and all integers are in numerical order. Furthermore, if the nth element in the list is an integer it must remain an integer, and if it is a word it must remain a word.

Input Specification:
The input will contain a single, possibly empty, line containing a space-separated list of strings to be sorted. Words will not contain spaces, will contain only the lower-case letters a-z. Integers will be in the range -999999 to 999999, inclusive. The line will be at most 1000 characters long.
 
Output Specification:
The program must output the list of strings, sorted per the requirements above. Strings must be separated by a single space, with no leading space at the beginning of the line or trailing space at the end of the line.
 
Sample Input 1:

1

Sample Output 1:

1
  

Sample Input 2:

car truck bus

Sample Output 2:

bus car truck
 

Sample Input 3:

8 4 6 1 -2 9 5

Sample Output 3:

-2 1 4 5 6 8 9


Sample Input 4:

car truck 8 4 bus 6 1

Sample Output 4:

bus car 1 4 truck 6 8



### 解答第一题

#### 简述题意:

设定一组正整数,个数为N\(N > 1,2 <= N <= 10^5\),将这组正整数分成两份,个数分别为N1,N2.再两个集合是A1,A2来存放,之后再获取A1,A2中的总和,赋值为S1,S2.要求当\|N1-N2\|最小时,\|S1-S2\|最大,并输出.所有的数字和结果小于2^31

#### 解题思路:

个数差最小只有两种情况,0和1.当N为偶数时,个数差为0,但N为奇数是,个数差为1.将一开始存放数据的集合进行排序,若按从大到小的方式,当N为偶数时,N1为N/2,当N为奇数时,N1为\(N/2 + 1\).分好后计算总和差即可

#### 答案代码:

	package com.cgtest.demo;

	import java.util.Scanner;
	import java.util.Vector;

	/**
	 * @author cg
	 *思路:个数差最小为0或1,按大小排序分个数后总和相减
	 *偶数为0,平分个数
	 *奇数为1,多取一位
	 *
	 */
	public class TestDemo1 {
		public static void main(String [] args){
			Hint hint = new Hint();
			int n = hint.showHint();
			hint.getValue(hint.sortVector(hint.getAllValue(n)));
		}
	}

	class Hint{
		Scanner scanner = new Scanner(System.in);
	
		public int showHint(){
			int n = -1;
			System.out.println("输入正整数的个数:");
			n = scanner.nextInt();
			return n; 
		}
	
		public Vector<Integer> getAllValue(int n){
			Vector<Integer> vector = new Vector<Integer>();
		
			for(int i = 0; i < n; i++){

				System.out.println("请输入第" + (i + 1) + "个数");
				int x = scanner.nextInt();
				vector.add(i, x);
			
			}
		
			for(int i = 0; i < vector.size(); i++){
				System.out.print(vector.get(i) + "----");
			}
		
			scanner.close();
		
			return vector;
		}
	
		public Vector<Integer> sortVector(Vector<Integer> vector){
		
			for(int i = 0; i < vector.size(); i++){
				for(int j = 0; j < vector.size() - 1; j++){
				
					if((int)vector.get(j) < (int)vector.get(j + 1)){
					
						int temp = (int)vector.get(j);
						vector.set(j, vector.get(j+1));
						vector.set(j+1, temp);
					}
				}
			}
			return vector;
		
		}
	
		public void getValue(Vector<Integer> vector){
			Vector<Integer> vector1 = new Vector<Integer>();
			Vector<Integer> vector2 = new Vector<Integer>();
		
			if(vector.size() % 2 == 0){
				System.out.println("个数差为0");
				oddSize(vector, vector1, vector2);
			}else{
				System.out.println("个数差为1");
				evenSize(vector, vector1, vector2);
			}
		
		}
		public void oddSize(Vector<Integer> vector,Vector<Integer> vector1,Vector<Integer> vector2){
		
			int n1 = 0,n2 = 0;
		
			for(int i = 0; i < vector.size() / 2; i++){
				vector1.add(i, vector.get(i));
			}
			for(int i = vector.size() / 2 ,j = 0; i < vector.size(); i++,j++){
				vector2.add(j, vector.get(i));
			}

			for (Integer integer : vector1) {
				n1 += integer;
			}
			System.out.println("n1=" + n1);
		
			for (Integer integer : vector2) {
				n2 += integer;
			}
		
			System.out.println(n1 - n2);
		
		}
		public void evenSize(Vector<Integer> vector,Vector<Integer> vector1,Vector<Integer> vector2){
		
			int n1 = 0,n2 = 0;
		
			for(int i = 0; i < (vector.size() + 1) / 2; i++){
				vector1.add(i, vector.get(i));
			}
			for(int i = (vector.size() + 1) / 2 ,j = 0; i < vector.size(); i++,j++){
				vector2.add(j, vector.get(i));
			}
		
			for (Integer integer : vector1) {
				n1 += integer;
			}
		
			for (Integer integer : vector2) {
				n2 += integer;
			}
		
			System.out.println(n1 - n2);
		}
	
	}

### 第二题:

#### 简述题意:

写一个小程序,实现输入一段字符串\(包括数字和字符\)后,按规则排序后输出
其规则是

* 数字与数字之间排序,单个字符串和字符串之间排序,
* 原本放数字的位置就放数字,原本放字符串的位置还是放字符串.
* 数字按大小排序,字符串依据首字母按26个字母表排序

#### 解题思路:

首先用一个集合存放整个字符串,实现单个字符串里的字符不分离.再将该集合中的单个字符串元素和数字元素分离,并获取在原集合存放的排列位置\(下标 + 1\)分别存放到两个二维集合中\(类似键值对的方式\),再操作这两个集合,实现排序,然后将排序后的两个二维集合整合,再排序,此时的排序则是按照存放集合中的原集合存放的排列位置排序,并输出,就可以了

#### 答案代码:

	package com.cgtest.demo2;

	import java.util.ArrayList;
	import java.util.Scanner;
	import java.util.Vector;
	import java.util.regex.Pattern;
	 
	/**
	 * @author cg
	 *思路:将输入的内容区分数字和字符串后分别存放到二维集合中,元素内容格式:\[原集合的位数(下标+1), value\],将其看做整体
	 *并在这两个集合元素中的value按规则排序
	 *排序完成后重新整合,并按位数排序
	 */
	public class TestDemo2 {
		public static void main(String [] args){
			Hint hint = new Hint();
			Vector<String> vector = hint.getInputStr();
			Vector<Vector<String>> vectorSortStr = hint.getStringIndexAndValue(vector);
			Vector<Vector<Integer>> vectorSortInt = hint.getIntIndexAndValue(vector);
			ArrayList<ArrayList<String>> arrayList = hint.produceNewString(vectorSortStr, vectorSortInt);
			hint.showConsole(arrayList);
		}
	}
	class Hint{
		public Vector<String> getInputStr(){
		
			Scanner scanner = new Scanner(System.in);
			Vector<String> vector = new Vector<String>();
			StringBuffer strBuffer = new StringBuffer();
			int n = 0;
		
			System.out.println("请输入内容:");
			String str = scanner.nextLine();
		
			char [] arrayStr = str.toCharArray();
		
			for(int i = 0; i < arrayStr.length; i++){
				String strC = String.valueOf(arrayStr[i]);
				if(strC.equals(" ")){
					vector.add(n, strBuffer.toString());
					strBuffer = new StringBuffer();
					n++;
				}else if(i == arrayStr.length - 1){
					strBuffer.append(strC);
					vector.add(n, strBuffer.toString());
				}else{
					strBuffer.append(strC);
				}
			}
			System.out.println("排序前:");
			for (String vectorItem : vector) {
				System.out.print(vectorItem + "\t");
			}
			System.out.println();
			scanner.close();
			return vector;
		}
	
		public Vector<Vector<String>> getStringIndexAndValue(Vector<String> vector){
			/*
			 *获取String类型的字符串和在原集合中的位数(下标+1)
			 *并返回
			 */
			Vector<Vector<String>> vectorStr = new Vector<Vector<String>>();
			Vector<Vector<String>> vectorSortStr = new Vector<Vector<String>>();
		
			for(int i = 0; i < vector.size(); i++){
				Vector<String> vector1 = new Vector<String>();
				if(!isNum(vector.get(i))){
					vector1.add("" + (i + 1));
					vector1.add(vector.get(i));
					vectorStr.add(vector1);
				}
			}
		
			vectorSortStr = getSortStringVector(vectorStr);
		
			return vectorSortStr;
		}
	
		public Vector<Vector<Integer>> getIntIndexAndValue(Vector<String> vector){
			/*
			 * 获取int类型的整数和在原集合中的位数(下标+1)
			 * 并返回
			 */
			Vector<Vector<Integer>> vectorInt = new Vector<Vector<Integer>>();
			Vector<Vector<Integer>> vectorSortInt = new Vector<Vector<Integer>>();
		
			for(int i = 0; i < vector.size(); i++){
				Vector<Integer> vector1 = new Vector<Integer>();
				if(isNum(vector.get(i))){
					vector1.add((i + 1));
					vector1.add(Integer.valueOf(vector.get(i)));
					vectorInt.add(vector1);
				}
			}
		
			vectorSortInt = getSortIntVector(vectorInt);
		
			return vectorSortInt;
		}
	
		public Vector<Vector<String>> getSortStringVector(Vector<Vector<String>> vector){
			/*
			 * 将字符串安字母表(ASII)表排序
			 * 并返回
			 */
		
			String sortArray[] = new String[vector.size()];
		
		
			for(int i = 0; i < vector.size(); i++){
					sortArray[i] = vector.elementAt(i).get(1);
			}
			for(int i = 0; i < sortArray.length - 1; i++){
				for(int j = 0; j < sortArray.length - i -1; j++){
					if(sortArray[j].toCharArray()[0] > sortArray[j + 1].toCharArray()[0]){
						String strTemp = sortArray[j];
						sortArray[j] = sortArray[j + 1];
						sortArray[j + 1] = strTemp;
					}
				}
			}

			for(int i = 0; i < vector.size(); i++){
			
				vector.elementAt(i).set(1, sortArray[i]);
			
			}
			return vector;
		}
		public Vector<Vector<Integer>> getSortIntVector(Vector<Vector<Integer>> vector){
			/*
			 * 将整数按大小排序
			 * 并返回
			 */
		
			int sortArray[] = new int[vector.size()];
		
			for(int i = 0; i < vector.size(); i++){
				sortArray[i] = vector.elementAt(i).get(1);
			}
			for(int i = 0; i < sortArray.length - 1; i++){
				for(int j = 0; j < sortArray.length - i -1; j++){
					if(sortArray[j] > sortArray[j + 1]){
						int strTemp = sortArray[j];
						sortArray[j] = sortArray[j + 1];
						sortArray[j + 1] = strTemp;
					}
				}
			}
			for(int i = 0; i < vector.size(); i++){
				vector.elementAt(i).set(1, sortArray[i]);
			}
			return vector;
		}
	
		public ArrayList<ArrayList<String>> produceNewString(Vector<Vector<String>> vectorStr, Vector<Vector<Integer>> vectorInt){
			/*
			 * 将两个集合整合并排序,返回
			 */
		
			ArrayList<ArrayList<String>> arrayListStr = new ArrayList<ArrayList<String>>();
		
			for(int i = 0; i < vectorStr.size(); i++){
				ArrayList<String> arrayList1 = new ArrayList<String>();
				for(int j = 0; j < vectorStr.elementAt(i).size(); j++){
					arrayList1.add(vectorStr.elementAt(i).get(j));
				}
				arrayListStr.add(arrayList1);
			}
		
			for(int i = 0; i < vectorInt.size(); i++){
				ArrayList<String> arrayList1 = new ArrayList<String>();
				for(int j = 0; j < vectorInt.elementAt(i).size(); j++){
					arrayList1.add(vectorInt.elementAt(i).get(j).toString());
				}
				arrayListStr.add(arrayList1);
			}
		
			for(int i = 0; i < arrayListStr.size(); i++){
				for(int j = 0; j < arrayListStr.get(i).size(); j++){
					for(int m = 0; m < arrayListStr.size() - i - 1; m++){
						if(Integer.parseInt("" + arrayListStr.get(m).get(0)) > Integer.parseInt("" + arrayListStr.get(m + 1).get(0))){
							ArrayList<String> temp = arrayListStr.get(m);
							arrayListStr.set(m, arrayListStr.get(m + 1));
							arrayListStr.set(m + 1, temp);
						}
					}
				}
			}
			return arrayListStr;
		}
	
		public boolean isNum(String str) {    
		    Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$");    
		    return pattern.matcher(str).matches();    
		  }  
	
		public void showConsole(ArrayList<ArrayList<String>> arrayList){
			System.out.println("排序后:");
			for (ArrayList<String> arraylist : arrayList) {
				System.out.print(arraylist.get(1) + "\t");
			}
		}
	
	}

在这里,提醒下自己,以后写完代码一定要测试完全\!\!多测试没错,而我,就是没测试完全,就将我的代码发给了hr,不仅没按规定时间完成,还发了个错误的代码,,,,,,能过才
而我的出错的地方就是没实现对负数的处理,要是输入的字符串中包含负数,
`isNum(String str)`
就不能识别负数,会将负数判定为字符串,,,,,
原先的

`isNum(String str)`

的代码如下:

	public boolean isNum(String str){
		/*
		* 不能判断负数,,,,,,
		*/
	 for (int i = str.length();--i>=0;){    
	   if (!Character.isDigit(str.charAt(i))){  
	    return false;  
	   	}  
	  }  
	 return true;  
	}

我的答案代码太过臃肿,我也完全没想到笔试题是关于算法的,我应聘的是java实习生啊,又不是C/C++语言,不过想想hr肯定是有他的目的的
代码太臃肿,还得多学习

