---
layout: post
title: "Python通过字符串查找包含该字符串的文件"
date: 2017-09-12 17:32:49
categories: python
tags: [python, codeing]
---

无聊很可怕，上班无聊也是一样。今天上班无聊翻自己做的笔记看，但是笔记太多，所以我就只想看包含一些已知内容的笔记，但又不知道存放在哪里，苦逼实在系统windows，要是linux直接用命令很快就找出来了，于是就打算用python写个脚本，顺便回顾一下python,好久没用了

<!-- more -->

### 实现描述
根据字符串查找，使用的是re模块中的findall方法，属于精确匹配，这样使得查找很尴尬，需要输入很准确的字符才能查找到文件，模糊匹配并没有去做，现在做的只要是为了实现我这很简单的需求....，通过精确匹配，将匹配结果未true的进行存放到字典中，并最终存放到列表中进行返回，之后直接遍历得到的列表来显示查找得到的结果
其中有一点，就是文件编码问题，由于笔记包含了各种字符，存放的编码格式有utf-8,ascii,gb2312等，所以再open的时候不能统一编码，所以这里是用chardet先判断下编码格式，然后返回给open使用


### 代码
这里贴代码

	import re
	import os
	import sys
	import time
	import chardet

	#describe: find files by string
	#author: cg

	def main():
		print("查找出该路径下所有包含此字符串的文件路径")
		while True:
			print("请输入文件路径('q to exit') :")
			inputFilePath = input('>')
			if (inputFilePath == 'q') or (inputFilePath == 'Q'):
				break
			elif inputFilePath != "":
				if os.path.exists(inputFilePath):
					print("请输入要查找的字符串 :")
					inputStr = input('>')
					if (inputStr == 'q') or (inputStr == 'Q'):
						break
					startTime = time.time()
					resultList = searchStr(inputStr, inputFilePath)
					useTime = round((time.time() - startTime), 4)
					showResult(resultList)
					print("查找所耗时间 %s s" %useTime)
					print("<-----查找完成----->")
					print('')
				else:
					print("文件路径[%s]不存在" %(inputFilePath))
					continue

	def searchStr(str1, path):
		listTotalFile = []
		dictTotalMsg = {}
		totalFile = 0
		for rootPathStr, dirNameLists, fileNameLists in os.walk(path, True):
			for fileName in fileNameLists:
				filePath = os.path.join(rootPathStr, fileName)
				fileEncoding = getFileEncode(filePath)
				with open(filePath, 'r', encoding=fileEncoding) as fileObj:
					totalNum = 0
					lineNum = 0
					dictTotal = {}
					listItemLine = []
					listItemCount = []
					while True:
						try:
							line = fileObj.readline()
							lineNum += 1
							if line:
								resultList = re.findall(str1, line)
								if len(resultList) > 0:
									totalNum += len(resultList)
									listItemLine.append(lineNum)
									listItemCount.append(len(resultList))
							else:
								break
						except Exception:
							print("error:")
							print("<<" + filePath + "读取失败 >>")
							continue

					if totalNum > 0:
						dictTotal['filePath'] = filePath
						dictTotal['totalCount'] = totalNum
						dictTotal['detailLine'] = listItemLine
						dictTotal['detailCount'] = listItemCount
						listTotalFile.append(dictTotal)
			totalFile += (len(fileNameLists))
		dictTotalMsg['findPath'] = path
		dictTotalMsg['findStr'] = str1
		dictTotalMsg['totalFileNum'] = totalFile
		dictTotalMsg['msg'] = listTotalFile
		return dictTotalMsg

	def getFileEncode(filePath):
		with open(filePath, 'rb') as fileObj:
			data = fileObj.read()
			fileDataDict = chardet.detect(data)
			return fileDataDict.get('encoding')
		
	def showResult(dictTotalMsg):
		listMsg = dictTotalMsg.get('msg')
		print("<-----查找结果----->")
		print("查找路径为: %s" %(dictTotalMsg.get('findPath')))
		print("查找的字符串为: %s" %(dictTotalMsg.get('findStr')))
		if len(listMsg) > 0:
			print("包含该字符串的文件路径及详情如下 :")
		else:
			print("抱歉 , 未查找到相应文件")
		for i in range(len(listMsg)):
			print("文件%s路径 : %s" %((i + 1), listMsg[i].get('filePath')))
			print("出现该字符串的总数 : %s" %(listMsg[i].get('totalCount')))
			print("出现该字符串的行数 : %s" %(listMsg[i].get('detailLine')))
			print("行数对应的出现次数 : %s" %(listMsg[i].get('detailCount')))
		print("总查找文件个数 : %s" %(dictTotalMsg.get('totalFileNum')))


	if __name__ == '__main__':
		main()
		

代码不多，也就100行，当然，功能也非常简单。

### 运行示例

![python-hwEncryption](/images/python/python-searchByStr.png)

看上面输出，还是有编码问题.....
