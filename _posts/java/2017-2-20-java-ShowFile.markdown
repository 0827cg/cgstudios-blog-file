---
layout: post
title: "Java遍历显示文件"
date: 2017-2-20 15:10:11
categories: java
tags: [java, codeing]
---

用来遍历文件夹内容的后树形显示内容的一段小代码
如下代码:

<!-- more -->

	import java.io.File;

	public class ShowFile {
		public static void main(String [] args){
			String filePath = "/home/cg/Work/Testnew/Java";
			printFile(new File(filePath), 1);
		}
		public static void printFile(File fileDir,int n){
			if(fileDir.isDirectory()){
				File nextDir [] = fileDir.listFiles();
				for(int i = 0;i < nextDir.length;i++){
					for(int j = 0;j < n;j++){
						System.out.print("| - - -");
					}
					System.out.println(nextDir[i].getName());
					if(nextDir[i].isDirectory()){
						printFile(nextDir[i],n + 1);
					}
				}
			}
		}
	}

这段代码将保存在上面的路径/home/cg/Work/Testnew/Java文件夹下,名字为ShowFile.java
之后在终端中编译运行,如下图:

![java-bookContorlSystem](/images/java/java-ShowFile.png)
