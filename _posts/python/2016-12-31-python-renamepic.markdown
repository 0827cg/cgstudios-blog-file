---
layout: post
title: "Python复制并重命名文件图片"
date: 2016-12-31 21:52:24
categories: python
tags: [python, codeing]
---

昨天用python写了一个小程序,用于复制并重命名电脑上所截屏存留的图片,linux系统的截屏多了,该文件夹打开速度也就不那么快了,而且linux截屏默认的图片命名格式很长,不方便用于上传到百度云,便写了个这个小脚本.

先不说别的,直接上的代码:

<!-- more -->

	import os
	import sys

	#author:cg错过   2016-12-30

	def main():

		#用于拷贝.py格式的文件到新的文件夹,并重命名
	    
		copyFile(reFileName())
	    

	def reFileName():
		#获取现在文件下的文件夹内的文件名,并返回
		#例如现在本文件存放在tt文件夹内,则此方法是获取tt文件夹内的所有文件的文件名(不包含文件夹),
	    index = 0
	    filePath = os.getcwd()
	    fileList = os.listdir(filePath)
	    fileNameList = []

		for fileElement in fileList:
			addFilePath = os.path.join(filePath, fileElement)
			if os.path.isdir(addFilePath):
			    continue

			fileType = os.path.splitext(fileElement)[1]

			if fileType == '.png':
			    
			    fileNameList.insert(index,fileElement)
			    index += 1
		
	    print(fileNameList)
	    return fileNameList


	def copyFile(fileNameList):

		#依照上一个方法的例子来说明
		#在tt文件夹下新建一个名为newFloder的文件夹
		#复制文件夹下的所有文件到tt文件夹下的newFolder文件夹下,并重命名

	    count = 0

	    filePath = os.getcwd()

		 if not os.path.exists('newFolder'):
			print("Create new folder, name is newFolder")
			os.mkdir('newFolder')
		    
		 else:
			print("Folder is exist,please rename a new folder")
			sys.exit()
	    

	    for fileElement in fileNameList:

		newFileName = str(count) + '.py'
		fileObj = open(str(fileElement), 'rb')
		
		fileObjContent = fileObj.read()        
		os.chdir('newFolder')

		newFileObj = open(newFileName, 'wb')
		newFileObj.write(fileObjContent)
		os.chdir(filePath)

		fileObj.close()
		newFileObj.close()

		count += 1

	    print("done")

	    
	if __name__ == '__main__':
	    main()

效果图:

![python-renamefile](/images/python/python-rename-1.png)

图中的Pictures文件夹就是存放截屏图片的,而newFloder-16-12-30文件夹是我手动重命名后的


首先先说一下这个脚本的大致实现方式:

方法reFileName\(\):
用于获取存放截屏图片的文件夹下的所有文件名,文件夹除外,获取文件名后并作为列表方式返回.方法copyFile\(list\)方法其参数是一个列表,将文件名列表作为参数传入方法,并使用二进制的读写方式来获取或写入图片二进制内容.后缀名为\.png.使用时直接将此脚本复制张贴到存放截屏图片的文件夹下,然后运行即可.

运行后脚本都将在先目录下新建一个名为"newFolder"的文件夹,并将重命名后的文件复制到此文件夹下

提前说明下,这个线程的脚本只是用来复制.png格式的图片,如要复制重命名其他格式文件,则需要更改代码中的`.png`这个代码即可
