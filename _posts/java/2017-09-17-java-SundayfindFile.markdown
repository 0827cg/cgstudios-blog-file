---
layout: post
title: "Java使用Sunday算法来查找文件"
date: 2017-09-17 15:43:07
categories: java
tags: [java, codeing, Sunday]
---

今天周末，出租屋无聊便来公司呆着。顺便看看Sunday算法
Sunday算法的查找匹配速率比KMP算法快，其匹配规则也简单易懂，其移动位数主要时参考与字符串中参加匹配的最末位字符的下一位字符，如果该字符并未在搜索串中出现，则将字符串指针移动到该字符的下一位字符，搜索串指针则归零，反之，如果参加匹配的最末位字符的下一位字符出现在搜索串中，则移动位数等于搜索串长度减去搜索串中第一次出现该字符的下标。

<!-- more -->

详情看末尾的引用，同样也谢谢这两篇文章的作者

### java实现代码


	public int sundaySearchStrByStr(String strTotal, String strSearch) {
		
		char charTotal [] = strTotal.toCharArray();
		char charSearch [] = strSearch.toCharArray();
		
		int t = 0;
		int s = 0;
		int existCount = 0;
		while(s < charSearch.length && t < charTotal.length) {
			
			if(charSearch[s] == charTotal[t]) {
				
				if((s + 1) != charSearch.length) {
					s++;
					t++;
				}else {
					existCount++;
					if(charTotal.length - (t + 1) >= charSearch.length) {
						s = 0;
						t++;
					}else {
						break;
					}
				}
			}else {
				int num = t + charSearch.length;
				int index = -1;
				if(num < charTotal.length) {
					
					for(int i = 0; i < charSearch.length; i++) {
						if(charTotal[num] == charSearch[i]) {
							index = i;
							break;
						}
					}
					if(index != -1) {
						t = t + (charSearch.length - index);
						s = 0;
						
					}else {
						t = num + 1;
						s = 0;
					}
				}else {
					break;
				}
			}
			if(t >= charTotal.length) {
				break;
			}
		}
		return existCount;
		
	}
	
整个Sunday算法的核心代码即while循环里面的代码，这里主要需注意字符串指针移动时的溢出问题，添加的条件即代码中的`num < charTotal.length`，满足此条件才能进行下一步，否则则跳出循环
另外，Sunday算法在while循环中多了一部for循环，其做的就是将那下一个字符与搜索串进行匹配，如果第一次就匹配成功，即break

### Sunday和KMP对比
就拿之前写的KMP算法代码来对比

#### KMP算法
如图
![java-KMP-searchFile](/images/java/kmp-search.png)

#### Sunday算法
如图
![java-Sunday-searchFile](/images/java/sunday-search.png)

所以总体来说，Sunday较KMP来说匹配速率更快，代码实现也更简单

引用:
1.[扩展 2：Sunday 算法-从头到尾彻底理解 KMP-极客学院wiki][]
2.[Sunday算法-温志泉博客][]

[扩展 2：Sunday 算法-从头到尾彻底理解 KMP-极客学院wiki]:http://wiki.jikexueyuan.com/project/kmp-algorithm/sunday.html
[Sunday算法-温志泉博客]:https://wenzhiquan.github.io/2016/05/28/java_pattern_string_match_sunday_algorithm/