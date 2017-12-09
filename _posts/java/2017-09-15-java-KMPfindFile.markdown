---
layout: post
title: "Java使用KMP算法来查找文件"
date: 2017-09-15 15:29:28
categories: java
tags: [java, codeing, python, KMP]
---

接上一篇文章，依据字符串来查找文件。当时使用Python来实现的，没使用啥算法，也就算是暴力匹配，查找速率很是慢。所以这次是使用KMP算法来实现。
首先要先了解KMP算法，记得大学的时候老师有讲过这个算法，可惜自己没好好听...于是网上找资料，主要就是看末尾引用的那篇文章，想了解KMP的倒可以看这篇，谢谢这位博主

<!-- more -->

### KMP算法

KMP算法有两种实现
1.基于部分匹配值表的实现
2.基于next数组的实现

KMP算法的第一种实现方式需要基于部分匹配值表，其大部分时候匹配移动的位数就是根据这个部分匹配值表来操作的，所以部分匹配值表对于这种KMP算法来说是很重要的。
这里我只实现了第一种方式，另一种基于next的方式并没去折腾。
这两种实现所遵循原则都一样，即摆脱每次只移动一位的匹配规则。

### KMP算法移动位数情况
KMP算法的移动方式都是将字符串固定，移动搜索串
假设有两个数组，搜索串:`searchStr[]`和字符串:`totalStr[]`,分别用下表`s`和`t`表示
无论`t`的值是多少，在当`searchStr[0]`与`totalStr[t]`相等时，即意味着在totalStr中有一个字符与searchStr的第一个字符相同，此时就需要确认下一个字符是否与`searchStr[1]`相同，那么将此刻不移动位数，将指针`从totalStr[t]`移到`total[t+1]`,对比是否和`search[1]`是否相等，如果相同那就将指针继续往后移动，如果不相同就该移动位数了，即移动`searchStr[]`这个数组，对于具体需要移动多少位，我想，如果使用最死的方法就是一位一位的移，但这样太浪费时间和资源了，这个时候就需要用到部分匹配值表，其移动位数值的计算公式如下

`移动位数 = 已经匹配的字符数 - 匹配不成功的字符数的上一位字符对应的部分匹配值`

注意，这都是移动搜索串，使字符串的`t++`
在前面的匹配都满足的时候，在当`searchStr[searchStr.length-1]`与`totalStr[t]`也相等时，即表示已经成功的在字符串中找着了搜索串，如果还需要继续匹配，即查找全部字符串，那么就需要将`searchStr[]`清零，`totalStr[]`下标`t+1`，继续匹配
当然，在继续匹配之前，可以判断下totalStr剩余的字符是否还够得完成一次匹配，如果不够，就可以直接跳出循环，结束匹配

### kmp算法代码实现

	while(s < searchChar.length && t < totalChar.length) {
		if(searchChar[s] == totalChar[t]) {
			if((s + 1) != searchChar.length) {
				s++;								//如果，且并为匹配完，即searchStr长度还有剩余
				t++;
			}else {
				existCount++;						//如果searchStr已经全部匹配完成，则当前文件存在的总个数自加
				if((totalChar.length - (t + 1)) >= searchChar.length) {
					s = 0;							//如果totalStr剩余的字符个数比searchStr字符个数多，即足够还能再匹配
					t++;
				}else {
					break;							//如果个数不够，不能完成一次匹配，则跳出循环
				}
			}
		}else if(s == 0){
			s = 0;									//如果是seasrchStr第一个字符成功匹配，则t自加,即searchStr移动一位。否则依据部分匹配表来移动位数
			t++;
		}else {
			s = s - (s - kmpTable[(s - 1)]);		//kmpTable是int类型的部分匹配值数组
		}
		if((t + 1) >= totalChar.length) {			//如果totalStr下表达到了totalStr的总字符数，则跳出循环
			break;
		}
	}

kmp算法大致类似，那么下面就需要知道部分匹配值表是如何通过代码得到的

### 部分匹配值表代码
其规则是，首先进行第一次拆分，即将一个字符串拆分，从首部开始拆分。例如字符串`ABC`,将其拆分成`A`,`AB`,`ABC`三个字符串
之后再将这三个字符串分别进行前缀，后缀拆分，例如将`ABC`拆分得到的前缀为`A`,`AB`,拆分得到的后缀为`C`,`BC`
然后就匹配`A`,`AB`和`C`,`BC`这四个字符串是否相等，如果有相等，则获取其字符串长度，如果有长度更待的字符串相等，则将前面获取的字符串长度替换成字符串长度更大的值
代码如下


	public int[] getKMPtable(String strInput) {
		
		/*
		 * 获取kmp的部分匹配数值表
		 * 但得先获取字符串所有可能长度的最大公告元素长度，将其存放到int数组中返回
		 */
		
		int intTablesLength = strInput.length();
		int kmp_table [] = new int [intTablesLength];
		
		for(int i = 0; i < strInput.length(); i++) {
			String strItem = strInput.substring(0, i + 1);
			int intMaxPublicNum = getMaxPublicNum(strItem);
			kmp_table [i] = intMaxPublicNum;
		}
		return kmp_table;
		
	}
	
	
	public int getMaxPublicNum(String strItem) {
		
		//获取前缀和后缀，并最终对比得到最大的公共元素长度,并返回
		
		int intMaxPublicNum = 0;
		int intItemLength = strItem.length();
		
		String strFront [] = new String [intItemLength - 1];
		String strBack [] = new String [intItemLength - 1];
		
		for(int i = 0; i < intItemLength - 1; i++) {
			strFront[i] = strItem.substring(0, i + 1);
		}
		for(int i = intItemLength; i > 1; i--) {
			strBack[intItemLength - i] = strItem.substring(i - 1, intItemLength);
		}
		
		int n = -1;
		for(int i = 0; i < intItemLength - 1; i++) {
			if(strFront[i].equals(strBack[i])) {
				n = i;
			}
		}
		if(n != -1) {
			intMaxPublicNum = strFront[n].length();
		}
		return intMaxPublicNum;
	}
	


上面代码中，方法`getKMPtable()`传入的参数即为搜索串，该方法将搜索串进行第一次拆分，将每一次拆分得到的字符串作为参数传入`getMaxPublicNum()`方法中，`getMaxPublicNum()`方法就是获取该字符串的最大公共字符串的长度，其做法就是将传入的字符串进行前缀后缀拆分，之后返回最大公共字符串长度，如果没有公共字符串则返回0
所有返回的最大公共字符串长度将被方法`getKMPtable()`操作存放到一个int类型的数组中，并最后返回这个数组
这个最大公共字符串长度对应的字符就是相同下表的搜索串的字符。

### java字符串搜索文件总体代码

	package com.cgtest.kmpsearch;

	import java.io.BufferedReader;
	import java.io.File;
	import java.io.FileNotFoundException;
	import java.io.FileReader;
	import java.io.IOException;
	import java.util.ArrayList;
	import java.util.HashMap;
	import java.util.Map;
	import java.util.Scanner;
	
	/**
	* @author cg
	* time: 2017-09-15
	* describe: use kmp algorithm to search files by string
	*/
	
	public class KMPsearchFile {
		
		public static void main(String [] args) {
			
			System.out.println("通过字符串来查找文件，使用全匹配的基于部分匹配表的KMP算法");
			Scanner scanner = new Scanner(System.in);
			while(true){
				System.out.println("请输入文件路径('q to exit') :");
				String strFilePath = scanner.nextLine();
				if(strFilePath.equals("q") || strFilePath.equals("Q")) {
					scanner.close();
					break;
				}else if(!strFilePath.equals("")){
					if(new File((strFilePath)).exists() && true) {
						System.out.println("请输入要查找的字符串 :");
						String strSearch = scanner.nextLine();
						if(strSearch.equals("q") || strSearch.equals("Q")) {
							scanner.close();
							break;
						}else {
							
							long startTime = System.currentTimeMillis();
							
							KMPself kmpself = new KMPself();
							int [] kmpTable = kmpself.getKMPtable(strSearch);
								
							Map<String, Object> mapTotalFile = kmpself.kmpSearchFileByStr(strFilePath, strSearch, kmpTable);
							double userTime = (System.currentTimeMillis() - startTime);
							kmpself.showResult(mapTotalFile);
							System.out.println("查找耗时:" + (userTime)/1000 + "s");
							}
							
						}
					}else {
						System.out.println("文件路径" + strFilePath + "不存在");
						continue;
					}
				}
			}
			
	}


	class KMPself{
		
		ArrayList<File> listFilesObj = null;
		
		@SuppressWarnings("unchecked")
		public void showResult(Map<String, Object> mapTotalFile) {
			
			ArrayList<Object> listMsg = (ArrayList<Object>) mapTotalFile.get("resultMsg");
			System.out.println("<-----查找结果----->");
			System.out.println("查找路径为:" + mapTotalFile.get("searchPath"));
			System.out.println("查找的字符串为:" + mapTotalFile.get("strSearch"));
			if(listMsg == null || listMsg.size() == 0) {
				System.out.println("已扫描文件个数 :" + mapTotalFile.get("fileNum"));
				System.out.println("已扫描字符个数 :" + mapTotalFile.get("totalCharNum"));
				System.out.println("抱歉 , 未查找到相应文件");
			}else {
				System.out.println("包含该字符串的文件路径及详情如下 :");
				for(int i = 0; i < listMsg.size(); i++) {
					Map<String, Object> mapItem = (Map<String, Object>) listMsg.get(i);
					System.out.println("文件" + (i + 1) + "路径 :" + mapItem.get("filePath"));
					System.out.println("出现该字符串的总数 :" + mapItem.get("totalCount"));
					System.out.println("出现该字符串的行数 :" + mapItem.get("lineNum"));
					System.out.println("行数对应的出现次数 :" + mapItem.get("lineExistCount"));
				}
				System.out.println("总查找文件个数 :" + mapTotalFile.get("fileNum"));
				System.out.println("总查找字符个数 :" + mapTotalFile.get("totalCharNum"));
				System.out.println("<-----查找完成----->");
			}
		}
		
		
		public ArrayList<File> getFiles(String strFilePath) {
			
			if(listFilesObj == null) {
				listFilesObj = new ArrayList<File>();
			}

			File fileObj = new File(strFilePath);
			
			if(fileObj.isDirectory()) {
				File fileNextDir [] = fileObj.listFiles();
				for(File fileItem : fileNextDir) {
					if(fileItem.isDirectory()) {
						getFiles(fileItem.getPath());
					}else {
						listFilesObj.add(fileItem);
					}
				}
			}else {
				listFilesObj.add(fileObj);
			}
			return listFilesObj;
		}
		
		
		public Map<String, Object> kmpSearchFileByStr(String strFilePath, String strSearch, int kmpTable []) {
			
			/*
			 * 使用kmp算法
			 * 通过字符串搜索文件，将搜索到的结果封装到map，list混合集合中，并最终返回一个map集合
			 */
			
			ArrayList<File> listFilesObj = getFiles(strFilePath);
			
			Map<String, Object> mapTotalFile = new HashMap<String, Object>();
			ArrayList<Object> listMsg = new ArrayList<Object>();
			int fileNum = 0;
			long totalCharNum = 0;
			
			for(File listItem : listFilesObj) {
				Map<String, Object> mapFile = new HashMap<String, Object>();
				ArrayList<Integer> listLineNum = new ArrayList<Integer>();
				ArrayList<Integer> listLineExistCount = new ArrayList<Integer>();
				int lineNum = 1;
				int existCount = 0;
				int totalCount = 0;
				try {
					BufferedReader buffererReader = new BufferedReader(new FileReader(listItem));
					String strLine = null;
					while((strLine = buffererReader.readLine()) != null) {
						existCount = kmpSearchStrByStr(strLine, strSearch, kmpTable);
						if(existCount != 0) {
							listLineNum.add(lineNum);
							listLineExistCount.add(existCount);
						}
						totalCharNum += strLine.length();
						totalCount += existCount;
						
						lineNum++;
					}
					buffererReader.close();
					if(totalCount != 0) {
						mapFile.put("filePath", listItem);
						mapFile.put("totalCount", totalCount);
						mapFile.put("lineNum", listLineNum);
						mapFile.put("lineExistCount", listLineExistCount);
					}
					if( mapFile.size() != 0) {
						listMsg.add(mapFile);
					}
					
				} catch (FileNotFoundException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				fileNum++;
			}
			if(listMsg.size() != 0) {
				mapTotalFile.put("resultMsg", listMsg);
			}
			mapTotalFile.put("searchPath", strFilePath);
			mapTotalFile.put("strSearch", strSearch);
			mapTotalFile.put("fileNum", fileNum);
			mapTotalFile.put("totalCharNum", totalCharNum);
			
			return mapTotalFile;
		}
		
		
		public int kmpSearchStrByStr(String totalStr, String strSearch, int kmpTable []) {
			/*
			 * 参数1为内容字符串
			 * 参数2为输入的搜索字符串即搜索串
			 * 参数3为输入的搜索字符串的部分匹配数值表，为int类型的一维数组
			 * 全匹配的基于部分匹配表的KMP算法
			 * 并不是基于next数组
			 * 
			 * 其返回值是当前字符串中有出现搜索串的个数
			 * 此时并无下标
			 * 
			 */
			char searchChar [] = strSearch.toCharArray();
			char totalChar [] = totalStr.toCharArray();
			
			int s = 0;
			int t = 0;
			//int index = -1;
			int existCount = 0;
			while(s < searchChar.length && t < totalChar.length) {
				if(searchChar[s] == totalChar[t]) {
					if((s + 1) != searchChar.length) {
						s++;
						t++;
					}else {
						existCount++;
						if((totalChar.length - (t + 1)) >= searchChar.length) {
							s = 0;
							t++;
						}else {
							break;
						}
					}
				}else if(s == 0){
					s = 0;
					t++;
				}else {
					s = s - (s - kmpTable[(s - 1)]);
				}
				if((t + 1) >= totalChar.length) {
					break;
				}
			}
			return existCount;
			
		}
		
		
		
		public int[] getKMPtable(String strInput) {
			
			/*
			 * 获取kmp的部分匹配数值表
			 * 但得先获取字符串所有可能长度的最大公告元素长度，将其存放到int数组中返回
			 */
			
			int intTablesLength = strInput.length();
			int kmp_table [] = new int [intTablesLength];
			
			for(int i = 0; i < strInput.length(); i++) {
				String strItem = strInput.substring(0, i + 1);
				int intMaxPublicNum = getMaxPublicNum(strItem);
				kmp_table [i] = intMaxPublicNum;
			}
			return kmp_table;
			
		}
		
		
		public int getMaxPublicNum(String strItem) {
			
			//获取前缀和后缀，并最终对比得到最大的公共元素长度,并返回
			
			int intMaxPublicNum = 0;
			int intItemLength = strItem.length();
			
			String strFront [] = new String [intItemLength - 1];
			String strBack [] = new String [intItemLength - 1];
			
			for(int i = 0; i < intItemLength - 1; i++) {
				strFront[i] = strItem.substring(0, i + 1);
			}
			for(int i = intItemLength; i > 1; i--) {
				strBack[intItemLength - i] = strItem.substring(i - 1, intItemLength);
			}
			
			int n = -1;
			for(int i = 0; i < intItemLength - 1; i++) {
				if(strFront[i].equals(strBack[i])) {
					n = i;
				}
			}
			if(n != -1) {
				intMaxPublicNum = strFront[n].length();
			}
			return intMaxPublicNum;
		}
		
		
	}


同时，我也对原先写的python代码进行了修改，使用KMP算法

### python实现KMP算法代码

其python实现的KMP算法核心代码如下

	def kmpSearchStrByStr(totalStr, strSearch, kmpTable):

		#kmp算法查找
		#返回字符串中包含搜索串的个数

		listSearch = list(strSearch)
		listTotal = list(totalStr)
		
		s = 0
		t = 0
		existCount = 0
		while((s < len(listSearch)) & (t < len(listTotal))):
			if(listSearch[s] == listTotal[t]):
				if((s + 1) != len(listSearch)):
					s+=1
					t+=1
				else:
					existCount+=1
					if((len(listTotal) - (t + 1)) >= len(listSearch)):
						s = 0
						t+=1
					else:
						break;
			elif(s == 0):
				s = 0
				t+=1
			else:
				s = s - (s - kmpTable[(s - 1)])
			if((t + 1) >= len(listTotal)):
				break
		#print(existCount)
		return existCount
		


	def getKMPtable(strSearch):

		#获取kmp的部分匹配数值表
		#但得先获取字符串所有可能长度的最大公告元素长度，将其存放到int数组中返回

		intTablesLength = len(strSearch)
		kmpTable = []

		for i in range(intTablesLength):
			strItem = strSearch[0 : i + 1]
			intMaxPublicNum = getMaxPublicNum(strItem)
			kmpTable.append(intMaxPublicNum)

		#print(kmpTable)
		return kmpTable


	def getMaxPublicNum(strItem):

		#获取前缀和后缀，并最终对比得到最大的公共元素长度,并返回
		
		intMaxPublicNum = 0
		intItemLength = len(strItem)

		listFront = []
		listBack = []

		for i in range(intItemLength - 1):
			listFront.append(strItem[0 : i + 1])

		for i in range(intItemLength, 1, -1):
			listBack.append(strItem[i - 1 : intItemLength])

		n = -1
		for i in range(intItemLength - 1):
			if(listFront[i] == listBack[i]):
				n = i
		if(n != -1):
			intMaxPublicNum = len(listFront[n])
			
		#print(intMaxPublicNum)
		return intMaxPublicNum

### python和java搜索对比
python实现的字符串搜索文件和java实现的字符串搜索文件，其运行速率对比还是很明显，估计问题就在python对文件编码格式上面，如图

![java-python-KMP-searchFile](/images/java/python-java-kmp-searchFile.png)


速率相差太大，估计就是代码的问题
java代码同样也是臃肿.....



引用: [算法系列之二十六--字符串匹配之KMP算法][]

[算法系列之二十六--字符串匹配之KMP算法]:http://blog.csdn.net/sunnyyoona/article/details/44004563