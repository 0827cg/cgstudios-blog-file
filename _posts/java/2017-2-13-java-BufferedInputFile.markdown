---
layout: post
title: "Java读取自己文件内容"
date: 2017-2-13 20:03:57
categories: java
tags: [java, codeing]
---

这里使用对文件进行缓冲,将得到的引用传给BufferedReader构造器,从而实现对文件的读取

代码如下:

<!-- more -->

	public class BufferedInputFile {
		public static String read(String fileName) throws IOException{
		
			BufferedReader br = new BufferedReader(new FileReader(fileName));
		
			String s = null;
		
			StringBuffer sb = new StringBuffer();
			while((s = br.readLine()) != null){
				sb.append(s + "\n");
			}
			br.close();
			return sb.toString();
		}
	
		public static void main(String [] args) throws IOException{
			System.out.println(read("BufferedInputFile.java"));
		}
	}

程序的输出则为该程序的源代码,此段程序的文件名为BufferedInputFile.java.

文件的运行需要在终端中使用命令运行,如若在Eclipse中运行,则会抛出FileNotFound异常

其中,BufferedReader中的readLine()方法会将换行符删掉,所以append方法内需要添加"\n"

运行如下图:

![java-BufferedInputFile](/images/java/java-BufferedInputFile.png)
