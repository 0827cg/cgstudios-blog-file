---
layout: post
title: "Python加密文本内容"
date: 2017-3-10 19:24:39
categories: python
tags: [python, codeing]
---

使用python来加密文本内容,这是我之前在图书馆偶然间看到了一本python加密算法的书,看到了就想了解下

现在做的只是整理重温,代码是以前的,笔记也是以前的,现在只不过是想放到博客里方便以后看

这里就直接写明我所在书上看到的简单的加密算法,
有三种:

* 反转加密法:
* 凯撒加密法:
* 换位加密法:

这三种都特别简单,只适合用来随便玩玩.
我想如果要是以后还记得的话,可以抽出时间来多深入的了解下关于这反面的知识.

<!-- more -->

ok,进入正题

### 反转加密法

#### 描述:

就是通过反向输出文本进行加密,就这样.这是最简单的加密方法

#### 加密方法:
	
	encryptionStr = ''
	i = len(message) - 1
	while i >= 0:
		encryption += message[i]
		i -= 1
	return encryptionStr

就这几行代码,将原文本反向输出,即完成了最简单的加密

#### 解密方法:

解密方法:再运行一次即可解密

就是这么简单直接,,,,

### 凯撒加密法

#### 描述:

通过数字秘钥匹配到字母表中,通过下标加上或减去秘钥得到新数字,以新数字为下标获取字符,用来替换原来的字符

#### 加密方法:

	letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
	encryptionStr = ''
	key = 
	message = message.upper()	
	for element in message:
		if element in letters:
			num = letters.find(element)
			num += key
			if num >= len(letters):
				num -= len(letters)
			elif num < 0:
				num += len(letters)
			encryptionStr += letters[num]
		else:
			encryptionStr += element
	return encryptionStr

这个就是根据一个数字,以这个数字作为秘钥,然后从文本中通过下标的只来跟秘钥做运算,得到的新的数字就为被代替成新字符

#### 解密方法:

将代码:

`num += key`

改成:

`num -= key`

### 换位加密法:

#### 描述:

通过数字秘钥更换文本内容中字符的顺序,得到新的排序

#### 加密方法:
	
	key = 
	encryptionStr = [''] * key
	for colNum in range(key):
		pointer = colNum
		while pointer < len(strMessage):
			encryptionStr[colNum] += strMessage[pointer]
			pointer += key
	return ''.join(encryptionStr)

这个比前两个稍微复杂一点,需要借助表格来填字符.

以画一个表格,key列,其行数就是需要能容纳下全部字符即可.加密过程就是将文本的内容按行(从第一行起,从左往右)依次填如表格,填完后,就按列读取,第一列从上往下读取,读完后再读第二列,,,,,依次读取下去,最后得到的就是密文

这个密文的加密程度稍微比前两个加密算法好点,但这跟秘钥值的关系较大

#### 解密方法:

	colNum = math.ceil(len(message) / key)
	rowNum = key
	shadeBoxNum = (colNum * rowNum) - len(message)
	decryptionStr = [''] * colNum
	pointerCol = 0
	pointerRow = 0
	for element in message:
		decryptionStr[pointerCol] += element
		pointerCol += 1
		if(pointerCol == colNum) or ((pointer == colNum - 1) and (pointerRow >= rowNum - shadeBoxNum)):
			pointerCol = 0
			pointerRow += 1
	return ''.join(decryptionStr)

解密就是跟加密反着来就是了,前面一样关键就是后面的读取方式,这时候就按照加密的填法读取了

### 实践:

我自己也实践了第三种算法.读取原文本文件后,执行加密算法后输出保存加密后的加密文件,

同样也有解密程序将加密文件解密后输出保存

* 加密文件: hwEncryptionFile.py
* 解密文件: hwDecryptionFile.py
* 原文件:ShowFile.java

这三个文件都存放在同一目录下

#### hwEncryptionFile.py

	import time
	import os
	import sys

	def main():
	    
	    print("使用换位算法加密同目录下的ShowFile.java文件")
	    inputFileName = 'ShowFile.java'
	    outputFileName = 'ShowFile-encrypted.java'
	    key = 8
	    
	    if not os.path.exists(inputFileName):
		print("%s 不存在" %(inputFileName))
		sys.exit()

	    if os.path.exists(outputFileName):
		print("This will overwrite the file %s.(c)Continue or (q)Quit ?" %(outputFileName))
		choiceStr = input('> ')
		if not choiceStr.lower().startswith('c'):
		    sys.exit()

	    #读取文件内容,并将'\n'替换成'|'        
	    fileContent = readFile(inputFileName)
	    
	    firstStartTime = time.time()

	    #将内容加密
	    encryptionFileContent = encryptionStr(key,fileContent)
	    
	    encryptionTime = round((time.time() - firstStartTime), 4) 

	    #将加密后的内容写入到新的文件中
	    writeFile(outputFileName,encryptionFileContent)

	    totalTime = round((time.time() - firstStartTime), 4)
	    print("Done encryption %s (%s characters)" %(inputFileName,len(encryptionFileContent)))
	    print("encrypted file is %s" %outputFileName)
	    print("enctyption time: %s seconds" %encryptionTime)
	    print("TotalTime %s seconds" %totalTime)

	def readFile(inputFileName):
	    
	    #读取文件内容,先将换行符替换成'|',再全部返回
	    
	    fileContent = ""
	    fileObj = open(inputFileName, 'r')
	    while True:
		line = fileObj.readline()
		if line:
		    line = line.replace("\n","|")
		    fileContent += line
		else:
		    break
	    fileObj.close()

	    return fileContent

	def encryptionStr(intKey,strContent):

	    #intKey:换位加密算法秘钥
	    #strContent:需要加密的内容(且不包含换行符)
	    #加密后返回

	    encryptionStrContent = [''] * intKey
	    for colNum in range(intKey):
		pointer = colNum
		while pointer < len(strContent):
		    encryptionStrContent[colNum] += strContent[pointer]
		    pointer += intKey

	    return ''.join(encryptionStrContent)

	def writeFile(outputFileName,fileContent):
	    fileObj = open(outputFileName,'w')
	    fileObj.write(fileContent)
	    fileObj.close()
	    

	if __name__ == '__main__':
	    main()

#### hwDecryptionFile.py

	import time
	import os
	import sys
	import math

	def main():

	    print("解密同目录下被encryptionFile.py加密的ShowFile-enctypted.txt文件")
	    inputFileName = 'ShowFile-encrypted.java'
	    outputFileName = 'ShowFile-decrypted.java'
	    key = 8
	    
	    if not os.path.exists(inputFileName):
		print("%s 不存在" %(inputFileName))
		sys.exit()

	    if os.path.exists(outputFileName):
		print("This will overwrite the file %s.(c)Continue or (q)Quit ?" %(outputFileName))
		choiceStr = input('> ')
		if not choiceStr.lower().startswith('c'):
		    sys.exit()
	    
	    #读取被加密文件的内容
	    fileContent = readFileContent(inputFileName)
	    
	    firstStartTime = time.time()
	    #解密文件内容
	    decryptionFileContent = decryptionStr(key,fileContent)
	    
	    #将解密后的内容中的'|'替换成'\n'
	    normalFileContent = regetNewLineSymbol(decryptionFileContent)
	    decryptionTime = round((time.time() - firstStartTime), 4)

	    #将解密得到的内容写到新的文件
	    writeFileContent(outputFileName,normalFileContent)
	    totalTime = round((time.time() - firstStartTime), 4)

	    print("Done decryption %s (%s characters)" %(inputFileName,len(normalFileContent)))
	    print("encrypted file is %s" %outputFileName)
	    print("enctyption time: %s seconds" %decryptionTime)
	    print("TotalTime %s seconds" %totalTime)


	def readFileContent(inputFileName):

	    #读取文件内容并返回
	    
	    fileObj = open(inputFileName, 'r')
	    fileContent = fileObj.read()
	    fileObj.close()
	    return fileContent

	def writeFileContent(outputFileName,fileContent):
	    fileObj = open(outputFileName,'w')
	    fileObj.write(fileContent)
	    fileObj.close()

	def decryptionStr(intKey,strContent):
	    
	    #intKey:换位加密算法秘钥-用来解密
	    #strContent:需要解密的内容(且不包含换行符)
	    #解密后返回

	    numOfCol = math.ceil(len(strContent) / intKey)

	    numOfRow = intKey

	    numOfShadedBox = (numOfCol * numOfRow) - len(strContent)

	    decryptionStr = [''] * numOfCol

	    pointerCol = 0
	    pointerRow = 0

	    for element in strContent:
		decryptionStr[pointerCol] += element
		pointerCol += 1

		if(pointerCol == numOfCol) or ((pointerCol == numOfCol - 1) and (pointerRow >= numOfRow - numOfShadedBox)):
		    pointerCol = 0
		    pointerRow += 1
		    
	    return ''.join(decryptionStr)
	    
	    
		    
	def regetNewLineSymbol(strContent):

	    #将含有'|'符号的内容还原成有'\n'的,并返回
	    
	    reStrContent = strContent.replace("|","\n")
	    return reStrContent


	if __name__ == '__main__':
	    main()

#### ShowFile.java

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

这里做测试的是java代码

测试如图:

![python-hwEncryption](/images/python/python-hwEncryption.png)



