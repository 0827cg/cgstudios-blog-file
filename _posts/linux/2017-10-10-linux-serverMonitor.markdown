---
layout: post
title: "python实现服务器监控脚本"
date: 2017-10-10 17:27:02
updated: 2017-10-11 18:54:07
categories: linux
tags: [linux, python]
---

这段时间都在写这个监控脚本，之前用shell写过一个，其实也可以用，但考虑到后期需要实现自动化运维，功能需要更加完善，于是就打算写一个python版的监控脚本

<!-- more -->

### 脚本概况

分模块来总结，分为
* 选择操作包
* 工具包
* 项目模块包

目前完成的这个脚本文件中，其目录结构文件如下

![python-monitor](/images/linux/python-monitor-2.png)

`monitor.py` 脚本入口
`monitorbin` 存放的operate.py这个文件及选择操作模块
`monitorbin.util`存放四个工具模块
`monitorbin.module` 存放需要监控的进程模块

### monitorbin包

#### operate.py

选择操作模块，被`monitor.py`调用执行
其代码如下

	from monitorbin.util.fileUtil import FileUtil
	from monitorbin.module.tomcatCheck import TomcatOperate
	from monitorbin.module.nginxCheck import NginxOperate
	from monitorbin.module.redisCheck import RedisOperate
	from monitorbin.util.emailUtil import EmailUtil

	#author: cg错过
	#time: 2017-09-30

	class Operate:

		#选择执行操作类

		def __init__(self):
			
			self.fileUtil = FileUtil()
		 
			dictNeedRunMsg = self.fileUtil.getNeedRunMsg()
			if(len(dictNeedRunMsg) > 1):
				self.runProcess(dictNeedRunMsg)
				emailUtil = EmailUtil(dictNeedRunMsg, self.fileUtil)
			elif(len(dictNeedRunMsg) == 1):
				self.fileUtil.writerContent("配置文件参数值不全", 'runErr')
			else:
				self.fileUtil.writerContent("配置文件读取失败", 'runErr')


		def runProcess(self, dictNeedRunMsg):

			#运行检测各个项目
			
			listKeys = dictNeedRunMsg.keys()
			for keyItem in listKeys:
				if(keyItem.find('tomcat') != -1):
					strTomcatPath = dictNeedRunMsg.get(keyItem)
					tomcatOperate = TomcatOperate(strTomcatPath, self.fileUtil.strMinTime, self.fileUtil)
					#print(strTomcatPath)
				if(keyItem.find('nginx') != -1):
					strNginxPath = dictNeedRunMsg.get(keyItem)
					nginxOperate = NginxOperate(strNginxPath, self.fileUtil.strMinTime, self.fileUtil)
					#print(strNginxPath)
				if(keyItem.find('redis') != -1):
					strRedisPath = dictNeedRunMsg.get(keyItem)
					redisOperate = RedisOperate(strRedisPath, self.fileUtil.strMinTime, self.fileUtil)
					#print(strRedisPath)

其中实例化FileUtil这个模块类，调用类中的方法`getNeedRunMsg()`或许根据配置文件过滤得到需要检测系统所需要的数据，为dict字典类型。得到数据后进行判断，其中，当长度大于1即可认为该`dictNeedRunMsg`或许到的数据是完全的，当长度等于1时，该dict中存放的值为`dictNeedRunMsg={'0': 'error'}`,表示数据不全，即配置文件中缺少配置运行所需的数据，使运行中止，另外就是当`dictNeedRunMsg`长度为0时，即未成功读取到配置文件中的数据，当然这种情况到这一步是不会发送的，因为在此之前在FileUtil这个模块类中已经进行了判断。可以看下面FileUtil这个模块类

### monitorbin.util包

#### fileUtil.py

文件及部分数据处理类,在脚本运行中只能存在一个该对象
代码如下

	import os
	import xml.dom.minidom
	import configparser
	from monitorbin.util.sysTime import RunTime

	#author: cg错过
	#time: 2017-09-30

	class FileUtil:

		#文件及部分数据处理类

		configurePath = 'conf'
		configureFileName = 'monitor.conf'

		def __init__(self):

			self.strLogPath = self.getLogPath()
			#strLogPath = self.getLogPath()
			self.setAttribute()

		def setAttribute(self):

			#设置一些属性
			
			runTime = RunTime()
			self.strDateTime = runTime.getDateTime()
			self.strHourTime = runTime.getHourTime()
			self.strMinTime = runTime.getMinTime()
			self.strHourMinTime = runTime.getHourMinTime()
			self.strNumSecondTime = runTime.getNumSecondTime()
			self.strNumHourTime = runTime.getNumHourTime()

			strlogContentSecondName = "monitor_content-" + self.strNumSecondTime + ".txt"
			strlogContentName = "monitor_content-" + self.strNumHourTime + ".txt"
			strlogErrName = "monitor_err-" + self.strNumHourTime + ".txt"
			strlogErrSecondName = "monitor_err-" + self.strNumSecondTime + ".txt"
			strRunErrName = "err-" + self.strNumSecondTime + ".txt"
			
			self.strlogContentSecondName = self.strLogPath + '/' + strlogContentSecondName
			self.strlogContentName = self.strLogPath + '/' + strlogContentName
			self.strlogErrName = self.strLogPath + '/' + strlogErrName
			self.strlogErrSecondName = self.strLogPath + '/' + strlogErrSecondName
			self.strRunErrPathName = self.strLogPath + '/' + strRunErrName
			

		def writerContent(self, strContent, strFileMark='Hour', whetherAdd=True):
			
			#strFileMark: 区分写入小时执行的文件还是分钟执行的文件
			#strContent: 写入文件的内容
			#whetherAdd: 是否在文件后面换行追加，默认True
			
			if(strFileMark == 'Hour'):
				if(whetherAdd & True):
					fileObj = open(self.strlogContentName, 'a')
					fileObj.write(strContent + "\n")
					fileObj.close()
				else:
					fileObj = open(self.strlogContentName, 'w')
					fileObj.write(strContent)
					fileObj.close()
			elif(strFileMark == 'Second'):
				if(whetherAdd & True):
					fileObj = open(self.strlogContentSecondName, 'a')
					fileObj.write(strContent + "\n")
					fileObj.close()
				else:
					fileObj = open(self.strlogContentSecondName, 'w')
					fileObj.write(strContent)
					fileObj.close()
			else:
				if(whetherAdd & True):
					fileObj = open(self.strRunErrPathName, 'a')
					fileObj.write(strContent + "\n")
					fileObj.close()
				else:
					fileObj = open(self.strRunErrPathName, 'w')
					fileObj.write(strContent)
					fileObj.close()

		def writerErr(self, strContent, strFileMark='Hour', whetherAdd=True):

			if(strFileMark == 'Hour'):
				if(whetherAdd & True):
					fileObj = open(self.strlogErrName, 'a')
					fileObj.write("\n" + strContent)
					fileObj.close()
				else:
					fileObj = open(self.strlogErrName, 'w')
					fileObj.write(strContent)
					fileObj.close()
			else:
				if(whetherAdd & True):
					fileObj = open(self.strlogErrSecondName, 'a')
					fileObj.write("\n" + strContent)
					fileObj.close()
				else:
					fileObj = open(self.strlogErrSecondName, 'w')
					fileObj.write(strContent)
					fileObj.close()
				
		

		def getXMLTagElementValue(self, strFilePath, strTagName, strTagElementName, intTagIndex):

			#获取xml文件指定标签的内容，返回一个字符串值
			#self: 对象本身
			#strTagName: 标签名字
			#strTagElementName: 标签中的元素名字
			#intTagIndex: 文件中出现该标签的序列号(即第几个，从0开始)
			
			confObj = xml.dom.minidom.parse(strFilePath)

			documentElementObj = confObj.documentElement
			listElementItem = documentElementObj.getElementsByTagName(strTagName)
			#按照顺序存放，文件内容中第一个出现该标签名字的就放在集合的下标为0的位置
			tagElement = listElementItem[intTagIndex]
			strTagElementValue = tagElement.getAttribute(strTagElementName)
			print(strTagElementName + "=" + strTagElementValue)
			return strTagElementValue


		def getConfFileValue(self, configParserObj, configureFileNameAndPath):

			#获取conf后缀的配置文件内容，返回一个字典
			#注释了不读取，值为空会读取
			#configParserObj: 读取配置文件的对象
			#configureFileNameAndPath: 配置文件路径
			#读取写入的key名字全部小写

			dictConfMsg = {}
			intMark = self.checkFileExists(configureFileNameAndPath)
			if(intMark == 1):
				configParserObj.read(configureFileNameAndPath)
				try:
					listSectionName = configParserObj.sections()
				except:
					self.writerContent("读取配置文件出错", 'runErr')
				else:
					for sectionItem in listSectionName:
						#print(sectionItem)
						listKeyName = configParserObj.options(sectionItem)
						#print(listKeyName)
						sectionObj = configParserObj[sectionItem]
						if(len(listKeyName) != 0):
							for keyItem in  listKeyName:
								valueItem = sectionObj[keyItem]
								if(valueItem == None):
									dictConfMsg[sectionItem] = listKeyName
								else:
									dictConfMsg[keyItem] = valueItem
						else:
							dictConfMsg[sectionItem] = ''
			#print(dictConfMsg)
			return dictConfMsg


		def readFileContent(self, inputFileName):

			#读取普通文件内容并返回
			
			fileObj = open(inputFileName, 'r')
			strFileContent = fileObj.read()
			fileObj.close()
			
			return strFileContent


		def initConfigureFile(self):
			
			#初始化配置文件

			strTomcatPath = "/home/liying/dev/tomcat-7.0.73"
			strNginxPath = "/usr/local/nginx"
			strRedisPath = "/home/liying/dev/redis-2.8.24"

			strServerName = "116"
			strUserName = "林繁"

			strLogPath = "logs"

			strSmtp_server = "smtp.qq.com"
			strEmail_sendAddr = "yakult-cg@qq.com"
			strEmail_sendPasswd = "lscgsbnjddtgdegc"
			
			strToEmail = "1542723438@qq.com"
			strToEmail2 = "1732821152@qq.com"

			strAuthor = "cg错过"
			strCreateTime = "2017-09-30"

			if not os.path.exists(self.configurePath):
				os.mkdir(self.configurePath)

			configureFileNameAndPath = self.configurePath + '/' + self.configureFileName

			config = configparser.ConfigParser(allow_no_value=True, delimiters=':')
			config.add_section('ProjectConfigure')
			config.add_section('UseConfigure')
			config.add_section('LogConfigure')
			config.add_section('EmailConfigure')
			config.add_section('ToEmail')
			config.add_section("Message")
			
			config.set('ProjectConfigure', 'tomcatpath', strTomcatPath)
			config.set('ProjectConfigure', 'nginxpath', strNginxPath)
			config.set('ProjectConfigure', 'redispath', strRedisPath)

			config.set('UseConfigure', 'servername', strServerName)
			config.set('UseConfigure', 'username', strUserName)
			
			config.set('LogConfigure', 'logpath', strLogPath)

			config.set('EmailConfigure', 'smtp_server', strSmtp_server)
			config.set('EmailConfigure', 'email_sendAddr', strEmail_sendAddr)
			config.set('EmailConfigure', 'email_sendPasswd', strEmail_sendPasswd)
			
			config.set('ToEmail', strToEmail)
			config.set('ToEmail', strToEmail2)

			config.set('Message', 'author', strAuthor)
			config.set('Message', 'createtime', strCreateTime)

			with open(configureFileNameAndPath, 'w') as configureFile:
				config.write(configureFile, space_around_delimiters=True)

			#print("done")


		def getNeedRunMsg(self):

			#根据配置文件的配置内容来选择代码执行
			#即从存放的字典中去除不需要检测运行的项目(未配置值的参数)，之后返回
			print("获取运行需要的配置数据")
			
			#fileUtil = FileUtil()
			dictNewConfMsg = {}
			dictConfMsg = self.readConfigureFile()
			intMark = self.checkConfMsg(dictConfMsg)
			if(intMark == 1):
				intTomcatMark = self.checkRunProject("tomcat", "tomcatpath", dictConfMsg)
				if(intTomcatMark == 0):
					del dictConfMsg['tomcatpath']

				intNginxMark = self.checkRunProject("nginx", "nginxpath", dictConfMsg)
				if((intNginxMark == 0)):
					del dictConfMsg['nginxpath']

				intRedisMark = self.checkRunProject("redis", "redispath", dictConfMsg)
				if(intRedisMark == 0):
					del dictConfMsg['redispath']
				dictNewConfMsg = dictConfMsg
			elif(intMark == 0):
				dictNewConfMsg['0'] = 'error'
				

			print("需要运行的有")
			print(dictNewConfMsg)
			return dictNewConfMsg


		def readConfigureFile(self):

			#读取脚本配置文件
			dictConfMsgTotal = {}
			configureFileNameAndPath = self.configurePath + '/' + self.configureFileName
			self.checkAndInitConfigure(configureFileNameAndPath)
			config = configparser.ConfigParser(allow_no_value=True, delimiters=':')
			dictConfMsg = self.getConfFileValue(config, configureFileNameAndPath)
			dictConfMsgTotal.update(dictConfMsg)
			if(len(dictConfMsgTotal)  == 0):
				self.writerContent("未获取到配置文件内容", 'runErr')

			return dictConfMsgTotal

		def checkConfMsg(self, dictConfMsg):

			#检测配置文件是否完全
			#其中日志路径和email值必须存在
			#所以这里只检查logpath和email
			#email仅包括smtp_server, email_sendaddr, email_sendpasswd
			
			intMark = -1
			if(len(dictConfMsg) != 0):
				for keyItem in dictConfMsg:
					if((keyItem == 'logpath') | (keyItem == 'smtp_server') | (keyItem == 'email_sendaddr')
					  | (keyItem == 'email_sendpasswd')):
						if(dictConfMsg.get(keyItem) == ''):
							strErr = ("未读取到%s配置参数的值，请修改配置文件" %(keyItem))
							self.writerContent(strErr, 'runErr')
							intMark = 0
							break
						else:
							intMark = 1
			else:
				self.writerContent("未读取到配置文件内容", 'runErr')
			return intMark
			   


		def checkRunProject(self, projectName, strKey, dictConfMsg):

			#根据配置文件的配置，判断并选择代码块来执行
			#如果返回值为1，则表示返回允许执行检测projectName这个项目

			intMark = -1
			if(strKey in dictConfMsg):
				if(dictConfMsg.get(strKey) != ''):
					intMark = 1
				else:
					intMark = 0
					strErr = ("未读取到%s配置参数,如需检测%s请修改配置文件" %(projectName, projectName))
					self.writerContent(strErr, 'runErr')
			return intMark


		def checkFileExists(self, configureFileNameAndPath):

			#检测配置文件是否存在，不存在则返回-1

			intMark = -1
			if(os.path.exists(configureFileNameAndPath)):
				intMark = 1

			return intMark

		def checkAndInitConfigure(self, configureFileNameAndPath):

			#检测并初始化配置文件

			intMark = self.checkFileExists(configureFileNameAndPath)
			if(intMark != 1):
				print("配置文件monitor.conf不存在,脚本自动创建并初始化")
				print("配置文件monitor.conf路径为" + self.configurePath + "/" + self.configureFileName)
				#self.writerContent("配置文件monitor.conf不存在,脚本自动创建并初始化", 'runErr')
				#strErr = ("配置文件monitor.conf路径为" + self.configurePath + "/" + self.configureFileName)
				#self.writerContent(strErr, 'runErr')
				self.initConfigureFile()

		def checkAndCreate(self, FileNameAndPath):

			#检测并创建日志文件路径

			intMark = self.checkFileExists(FileNameAndPath)
			if(intMark != 1):
				print("配置的日志文件夹路径不存在，脚本执行自动创建")
				#self.writerContent("配置的日志文件夹路径不存在，脚本执行自动创建", 'runErr')
				os.mkdir(FileNameAndPath)

		def getLogPath(self):
			
			#获取日志文件配置
			#需要运行的项目字典,即过滤完后的字典
			#dictConfMsg = self.getNeedRunMsg()
			#strLogPath = dictConfMsg.get('logpath')
			strLogPath = ''
			configureFileNameAndPath = self.configurePath + '/' + self.configureFileName
			self.checkAndInitConfigure(configureFileNameAndPath)
			config = configparser.ConfigParser(allow_no_value=True, delimiters=':')
			config.read(configureFileNameAndPath)
			if(config.has_section('LogConfigure')):
				strLogPath = config['LogConfigure']['logpath']
				self.checkAndCreate(strLogPath)
			else:
				print("配置文件内容缺少日志配置参数")
				#self.writerContent("配置文件内容缺少日志配置参数", 'runErr')
			return strLogPath
			

		def reWriterForEmail(self, listSendContent, dictEmailMsg):

			#重构需要邮件发送的内容，并设置对应主题，并返回list
			#重构后的list数据集合格式：无错误日志list长度为3，有错误日志list长度为4
			#list[0]: Hour或者Second或者no
			#list[1]: 邮件主题
			#list[2]: 需要发送的邮件内容(已重构的内容)
			#list[3]: 错误日志的相对路径地址

			strNewContent = ''
			strContent = listSendContent[1]

			strServerName = 'none'
			strUserName = 'cg'
			for keyItem in dictEmailMsg:
				if((keyItem == 'servername') | (keyItem == 'username')):
					if(keyItem == 'servername'):
						strServerName = dictEmailMsg.get('servername')
					else:
						strUserName = dictEmailMsg.get('username')
				else:
					continue

			strContentLine = "===================="

			strNewContent = strContent[:0] + strContentLine + "\n" + strContent[0:]
			strNewContent = (strNewContent + "\n" + strContentLine + "\n" + "---" +
							 strUserName + "\n" + "---" + self.strDateTime)

			listSendContent[1] = strNewContent
			if(listSendContent[0] == 'Hour'):
				strSubject = strServerName + "今日" + self.strHourTime + "时执行结果"
				listSendContent.insert(1, strSubject)
			elif(listSendContent[0] == 'Second'):
				strSubject = strServerName + "今日" + self.strHourMinTime + "时检测到异常"
				listSendContent.insert(1, strSubject)        

			else:
				strSubject = strServerName + "脚本运行异常"
				listSendContent.insert(1, strSubject)
			print("已重构......")
			print(listSendContent)
			return listSendContent

这个`fileUtil.py`文件中，功能包括配置文件的读取,判断,过滤以及日志文件的读取写入即邮件内容的操作，这是脚本的一个重要模块。写的时候本来时将文件处理和数据处理分开来写的，但写到中途又合并了，因为这是国庆放假前写的代码，国庆这8天假都没碰这代码，从昨天才开始继续写.....

#### process.py

封装了调用了`subprocess.Popen`模块，实现python操作linux命令。通过python运行linux命令，截获两种输出`stdout`和`stderr`，在我看来这可分为有持续输出和无持续输出，区分这个时避免在持续输出时因存放量过大而发生阻塞，具体可参考[这篇文章][]及[官方文档][],当然，这也有[中文文档][]
代码如下

	import subprocess

	#author: cg错过
	#time: 2017-09-30

	class ProcessCL:

		#python操作Linux模块

		def getResultAndProcess(self, strCL):

			#
			#获取无持续输出的命令操作后的结果
			#获取正常输出和错误输出，存放到dict中返回
			#strCL: 操作的命令
			
			dictResult = {}
			strOut = ''
			strErr = ''
			subObj = subprocess.Popen(strCL, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True,
									  universal_newlines=True)
			returnCode = subObj.poll()
			while returnCode is None:
				stdout, stderr = subObj.communicate()
				returnCode = subObj.poll()
				strOut += stdout
				strErr += stderr
			dictResult['stdout'] = stdout
			dictResult['stderr'] = stderr
			
			return dictResult


		def getContinueResultAndProcess(self, strCL):

			#获取有持续输出的程序的结果，将其分类(stdout和stderr)
			#但检测到无输出后就退出
			#strCL: 操作的命令
			
			strOut = ''
			strErr = ''
			dictResult = {}
			subObj = subprocess.Popen(strCL, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True,
									  universal_newlines=True)
			returnCode = subObj.poll()
			while returnCode is None:
				lineOut = subObj.stdout.readline()
				lineErr = subObj.stdout.readline()
				returnCode = subObj.poll()
				lineOut = lineOut.strip()
				lineErr = lineErr.strip()

				strOut += lineOut
				strErr += lineErr
				if((lineOut == '') | (lineErr == '')):
					break
				
			dictResult['stdout'] = strOut
			dictResult['stderr'] = strErr

			return dictResult

上面的代码中，`getResultAndProcess()`这个方法虽然是针对无持续输出的命令，但依然是加上了`strOut += stdout`这个输出追加方式。因为如若不加这个，在当连续两次调用这个方法时，会出现bug，如下

	dictResult['stdout'] = stdout
	UnboundLocalError: local variable 'stdout' referenced before assignment

即命令行执行时可能stdout和stderr都无值，都未给进行声明并赋值，所以导致后面添加到dict的时候就会出错，这个错误就是在变量赋值之前被引用

#### emailUtil.py
邮件发送模块
代码如下

	import sys
	from smtplib import SMTP_SSL
	from email.mime.text import MIMEText
	from email.mime.multipart import MIMEMultipart
	from email.header import Header

	#author: cg错过
	#time: 2017-09-30

	class EmailUtil:

		#发送邮件模块

		def __init__(self, dictNeedRunMsg, fileUtilObj):

			#dictNeedRunMsg:存放从配置文件中读取到的数据，其数据是本次检测运行所需要的数据
			#fileUtilObj: FileUtil的对象(脚本从运行到结束都只有这一个FileUtil对象)
			#其中的key有
			#smtp_server:
			#mail_sendAddr:
			#mail_sendPasswd:
			#mail_toAddr:

			self.fileUtilObj = fileUtilObj
			dictEmailMsg = self.getForEmailMsg(dictNeedRunMsg)
			listEmailContentMsg = self.checkAndGetForEmailListMsg()

			strServerName = dictEmailMsg.get('servername')
			strUserName = dictEmailMsg.get('username')
			listNewEmailContentMsg = self.fileUtilObj.reWriterForEmail(listEmailContentMsg, dictEmailMsg)
			
			self.choiceSend(dictEmailMsg, listNewEmailContentMsg)
			
		def getForEmailMsg(self, dictNeedRunMsg):

			#将从配置文件的完全读取到的数据中，抽取出发送邮件需要的数据，存放为dict类型并返回
			#dictNeedRunMsg: 存放从配置文件中读取到的数据，其数据是本次检测运行所需要的数据
			#因为有可能并不是所有项目都配置了需要运行

			dictMsgForEmail = {}
			for keyItem in dictNeedRunMsg:
				if((keyItem == 'email_sendaddr') | (keyItem == 'email_sendpasswd') |
				   (keyItem == 'smtp_server') | (keyItem == 'ToEmail') | (keyItem == 'logpath') |
				   (keyItem == 'servername') | (keyItem == 'username')):
					dictMsgForEmail[keyItem] = dictNeedRunMsg.get(keyItem)
		
			return dictMsgForEmail

		def checkAndGetForEmailListMsg(self):

			#检查是否存在日志，并读取该日志进行返回
			#返回格式:如无错误日志list长度为2，有错误日志list长度为3
			#list[0]: Hour或者Second或者no
			#list[1]: 需要发送的邮件内容(仅运行日志)
			#list[2]: 错误日志的相对路径地址
			
			#产生的日志文件中，在每分钟和每小时的两种情况中，只会运行一种情况
			#也就是只会产生一个日志
			
			listSendContent = []
			intExistsContent = self.fileUtilObj.checkFileExists(self.fileUtilObj.strlogContentName)
			intExistsContentS = self.fileUtilObj.checkFileExists(self.fileUtilObj.strlogContentSecondName)
			if(intExistsContent == 1):
				listSendContent.append('Hour')
				print(self.fileUtilObj.strlogContentName)
				strContent = self.fileUtilObj.readFileContent(self.fileUtilObj.strlogContentName)
				listSendContent.append(strContent)
				
				intExistsErr = self.fileUtilObj.checkFileExists(self.fileUtilObj.strlogErrName)
				if(intExistsErr == 1):
					print("每小时，有错误，附件")
					listSendContent.append(self.fileUtilObj.strlogErrName)
				else:
					print("每小时，无错误")
			elif(intExistsContentS == 1):
				listSendContent.append('Second')
				print(self.fileUtilObj.strlogContentSecondName)
				strContentS = self.fileUtilObj.readFileContent(self.fileUtilObj.strlogContentSecondName)
				listSendContent.append(strContentS)
				
				intExistsErrS = self.fileUtilObj.checkFileExists(self.fileUtilObj.strlogErrSecondName)
				if(intExistsErrS == 1):
					print("每分钟，有错误，附件")
					listSendContent.append(self.fileUtilObj.strlogErrSecondName)
				else:
					print("每分钟，无错误")

			else:
				listSendContent.append('no')
				strContent = "未产生日志文件"
				self.fileUtilObj.writerContent("未产生日志文件", 'runErr')
				listSendContent.append(strContent)
			print("未重构......")
			print(listSendContent)

			return listSendContent


		def choiceSend(self, dictEmailMsg, listEmailContent):

			#选择发送类型(有无附件)
			#dictEmailMsg: 发送和接受邮件账户，及smtp服务器地址
			#listEmailContent: 发送的邮件内容，主题，附件

			strSmtpServer = dictEmailMsg.get('smtp_server')
			strSendAddr = dictEmailMsg.get('email_sendaddr')
			strPasswd = dictEmailMsg.get('email_sendpasswd')
			listToAddr = dictEmailMsg.get('ToEmail')
			strSubject = listEmailContent[1]
			strContent = listEmailContent[2]

			if(len(listEmailContent) == 3):
				if(listEmailContent[0] != 'no'):
					self.sendEmailByString(strSmtpServer, strSendAddr, strPasswd,
								   listToAddr, strSubject, strContent)
				else:
					self.fileUtilObj.writerContent("邮件未发送", 'runErr')
			else:
				strErrFilePath = listEmailContent[3]
				self.sendEmailByStringAndFile(strSmtpServer, strSendAddr, strPasswd,
								   listToAddr, strSubject, strContent, strErrFilePath)
			
			#print(self.fileUtilObj.strlogContentSecondName)


		def sendEmailByString(self, strSmtpServer, strSendAddr, strPasswd,
							  listToAddr, strSubject, strContent):

			#用字符串来发送邮件
			#strSmtpServer: smtp服务器地址
			#strSendAddr: 邮件发送地址
			#strPasswd: 发送地址的登陆授权码
			#listToAddr: 接受邮件的地址，为list集合
			#strSubject: 邮件主题
			#strContent: 邮件内容字符串类型
			#message对象中的三个key('From','To','Subject')最好都要有，不然容易被识别为垃圾邮件
			
			#mail_port = '465'
			
			message = MIMEText(strContent, "plain", "utf-8")
			message['Subject'] = Header(strSubject, 'utf-8')
			message['From'] = Header('monitor<%s>' % strSendAddr, 'utf-8')
			message['To'] = Header('monitor.admin', 'utf-8')

			try:
				smtpObj = SMTP_SSL(strSmtpServer)
				#smtpObj.set_debuglevel(1)
				smtpObj.ehlo(strSmtpServer)
				smtpObj.login(strSendAddr, strPasswd)
				if(len(listToAddr) > 0):
					smtpObj.sendmail(strSendAddr, listToAddr, message.as_string())
					smtpObj.quit()
				else:
					self.fileUtilObj.writerContent("接收邮件地址为空", 'runErr')
				#self.fileUtilObj.writerContent("邮件发送成功")
			except:
				print(sys.exc_info()[0])
				self.fileUtilObj.writerContent("邮件发送失败", 'runErr')


		def sendEmailByStringAndFile(self, strSmtpServer, strSendAddr, strPasswd,
							  listToAddr, strSubject, strContent, strErrFilePath):

			#发送有附件的邮件
			#strSmtpServer: smtp服务器地址
			#strSendAddr: 邮件发送地址
			#strPasswd: 发送地址的登陆授权码
			#listToAddr: 接受邮件的地址，为list集合
			#strSubject: 邮件主题
			#strContent: 邮件内容字符串类型
			#message对象中的三个key('From','To','Subject')最好都要有，不然容易被识别为垃圾邮件
			
			#mail_port = '465'

			message = MIMEMultipart()
			message['Subject'] = Header(strSubject, 'utf-8')
			message['From'] = Header('monitor<%s>' % strSendAddr, 'utf-8')
			message['To'] = Header('monitor.admin', 'utf-8')

			message.attach(MIMEText(strContent, 'plain', 'utf-8'))

			annexFile = MIMEText(open(strErrFilePath, 'rb').read(), 'base64', 'utf-8')
			annexFile["Content-Type"] = 'application/octet-stream'
			annexFile["Content-Disposition"] = 'attachment; filename="err_logs.txt"'
			message.attach(annexFile)

			try:
				smtpObj = SMTP_SSL(strSmtpServer)
				#smtpObj.set_debuglevel(1)
				smtpObj.ehlo(strSmtpServer)
				smtpObj.login(strSendAddr, strPasswd)
				if(len(listToAddr) > 0):
					smtpObj.sendmail(strSendAddr, addrItem, message.as_string())
					smtpObj.quit()
				else:
					self.fileUtilObj.writerContent("接受邮件地址为空", 'runErr')
				#self.fileUtilObj.writerContent("附件邮件发送成功")
			except:
				print(sys.exc_info()[0])
				self.fileUtilObj.writerContent("附件邮件发送失败", 'runErr')


具体看代码中的注释即可

#### sysTime.py

时间模块
代码如下

	import time

	#author: cg错过
	#time: 2017-09-30

	class RunTime:

		#时间模块

		def getTime(self, strFormat):

			#按照格式获取时间

			nowTime = time.localtime()
			strFormatTime = time.strftime(strFormat, nowTime)
			return strFormatTime

		def getDateTime(self):
			return self.getTime("%Y-%m-%d %H:%M:%S")

		def getNumSecondTime(self):
			return self.getTime("%Y%m%d%H%M%S")

		def getNumHourTime(self):
			return self.getTime("%Y%m%d%H")

		def getMinTime(self):
			return self.getTime("%M")

		def getHourTime(self):
			return self.getTime("%H")

		def getHourMinTime(self):
			return self.getTime("%H%M")

### monitorbin.module包

#### tomcatCheck.py

tomcat检测模块
代码如下

	import subprocess
	import os
	from monitorbin.util.process import ProcessCL

	#author: cg错过
	#time: 2017-09-30

	class TomcatOperate:

		#tomcat检测模块
		#需要配合前面分时间段来运行
		#即至少要有个对tomcat进行所有全部的检测和部分检测两种功能
		#全检不提供脚本操作功能，例如自启。全检后不管是否正常都将发送邮件
		#部检提供自启，自启后只有检测到不正常才发送邮件

		def __init__(self, strTotalPath, intDateMin, fileUtilObj):
			
			#strTotalPath: tomcat的安装文件根目录的上一级目录
			#intDateMin: 当前运行脚本的分钟数
			#fileUtilObj: FileUtil的对象(脚本从运行到结束都只有这一个FileUtil对象)

			self.fileUtil = fileUtilObj
			self.intDateMin = intDateMin
			self.strTotalPath = strTotalPath
			#self.fileUtil.writerContent("你好")
			intCheckResult = self.fileUtil.checkFileExists(self.strTotalPath)
			if(intCheckResult == 1):
				self.checkTomcat()
			else:
				self.fileUtil.writerContent("配置的tomcat路径不存在", 'runErr')
		
		def checkTomcat(self):

			#检测tomcat

			strTomcatStatus = self.getTomcatStatus()
			dictTomcatMsg = self.getTomcatMsg(self.strTotalPath)
			listTomcatName = dictTomcatMsg.get('tomcatName')
			listTomcatPort = dictTomcatMsg.get('tomcatPort')
			listTomcatPath = dictTomcatMsg.get('tomcatPath')
			
			if(self.intDateMin == 30):
				for i in range(len(listTomcatPort)):
					intMark = self.checkTomcatStatusByPort(i, listTomcatName, listTomcatPort, strTomcatStatus)
					if(intMark == 1):
						#print("查看日志")
						#self.fileUtil.writerContent("查看日志")
						self.checkTomcatLogStatusByTomcatName(i, listTomcatName, listTomcatPort)
			else:
				for i in range(len(listTomcatPort)):
					#print(i)
					intMark = self.checkTomcatStatusByPort(i, listTomcatName, listTomcatPort,
														   strTomcatStatus, 'Second')
					if(intMark != 1):
						#print("重启")
						self.tryStartTomcat(i, listTomcatPath, listTomcatName)

						
		def getTomcatStatus(self):
			
			#获取进程中的tomcat
			tomcatStatusCL = "ps -ef | grep tomcat"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(tomcatStatusCL)
			strTomcatStatus = dictResult.get('stdout')
			return strTomcatStatus


		def checkTomcatStatusByPort(self, intIndex, listTomcatName, listTomcatPort, strTomcatStatus,
									strFileMark='Hour'):

			#检测tomcat是否运行，在运行返回1
			#intIndex: 要检测的tomcat所在dictTomcatMsg中tomcatName的下标
			#dictTomcatMsg: 存放tomcat文件名,端口号和对应路径,为字典类型包含列表

			intMark = -1
			
			#listTomcatName = dictTomcatMsg.get('tomcatName')
			#listTomcatPort = dictTomcatMsg.get('tomcatPort')

			if(strTomcatStatus.find(listTomcatName[intIndex]) != -1):
				intMark = 1
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent(("%s在运行" %(listTomcatName[intIndex])), 'Hour', False)
				#else:
					#self.fileUtil.writerContent(("%s在运行" %(listTomcatName[intIndex])), 'Second')
			else:
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent(("%s未运行" %(listTomcatName[intIndex])))
				else:
					self.fileUtil.writerContent(("%s未运行" %(listTomcatName[intIndex])), 'Second')
					#print("%s未运行" %(listTomcatName[intIndex]))

			return intMark


		def checkTomcatLogStatusByTomcatName(self, intIndex, listTomcatName, listTomcatPort):

			#检测tomcat日志输出是否正常，正常返回1
			#intIndex: tomcat所在dictTomcatMsg中tomcatName的下标，一般检测在运行的tomcat
			#dictTomcatMsg: 存放tomcat文件名,端口号和对应路径,为字典类型包含列表

			intMark = 1

			#listTomcatName = dictTomcatMsg.get('tomcatName')
			#listTomcatPort = dictTomcatMsg.get('tomcatPort')
			strOperateTomcatPath = listTomcatPath[intIndex]

			checkLogCL = "tail -n 200 " + strOperateTomcatPath + "/logs/catalina.out"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(checkLogCL)
			strOut = dictResult.get('stdout')
			if(strOut.find("exception") != -1):
				intMark = -1
				#print("%s日志输出异常" %(listTomcatName[intIndex]))
				self.fileUtil.writerContent(("%s日志输出异常" %(listTomcatName[intIndex])))
			else:
				self.fileUtil.writerContent(("%s日志输出正常" %(listTomcatName[intIndex])))
				#print("%s日志输出正常" %(listTomcatName[intIndex]))
				self.fileUtil.writerErr(strOut)
			return intMark
			


		def tryStartTomcat(self, intIndex, listTomcatPath, listTomcatName):

			#启动未运行的tomcat，启动成功返回1
			#intIndex: 未运行的tomcat所在dictTomcatMsg中tomcatName的下标
			#dictTomcatMsg: 存放tomcat文件名,端口号和对应路径，为字典类型包含列表

			intMark = -1

			#listTomcatPath = dictTomcatMsg.get('tomcatPath')
			#listTomcatName = dictTomcatMsg.get('tomcatName')
			#print("脚本尝试将其启动....")
			self.fileUtil.writerContent("脚本尝试将其启动....", 'Second')
			strOperateTomcatPath = listTomcatPath[intIndex]
			tryStartTomcatCL = strOperateTomcatPath + "/bin/./catalina.sh start"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(tryStartTomcatCL)
			strOut = dictResult.get('stdout')
			strErr = dictResult.get('stderr')
			if(strOut != ''):
				if((strOut.find('Tomcat started') != -1) & (strErr == '')):
					#print("%s已被脚本启动成功" %(listTomcatName[intIndex]))
					self.fileUtil.writerContent(("%s已被脚本启动成功" %(listTomcatName[intIndex])),
												'Second')
					intMark = 1
				else:
					#print("脚本启动%s未成功,请手动启动" %(listTomcatName[intIndex]))
					self.fileUtil.writerContent(("脚本启动%s未成功,请手动启动" %(listTomcatName[intIndex])),
												'Second')
					self.fileUtil.writerErr(strErr, 'Second')
			else:
				#print("%s启动命令未执行" %(listTomcatName[intIndex]))
				self.fileUtil.writerContent(("%s启动命令未执行,请手动执行" %(listTomcatName[intIndex])), 'Second')
				#print(strErr)
				self.fileUtil.writerErr(strErr, 'Second')
			return intMark


		def getTomcatMsg(self, strTotalPath):
			
			#根据存放多个tomcat的文件路径来查找多少个tomcat
			#获取tomcat安装文件的名称和对应的端口
			#通过读取tomcat配置文件，以此来获得端口号
			#返回一个字典，存放tomcat安装文件名和对应端口号
			
			dictTomcatMsg = {}
			listTomcatName = []
			listMsgName = os.listdir(strTotalPath)
			for item in listMsgName:
				nextPath = (strTotalPath + '/' +  item)
				if(os.path.isdir(nextPath)):
					confPath = (nextPath + '/conf/server.xml')
					if((item.find("tomcat") != -1) & (os.path.exists(confPath))):
						listMsgName.remove(item)
						listTomcatName.append(item)

			#fileUtil = FileUtil()
			listTomcatPort = []
			listTomcatPath = []
			for item in listTomcatName:
				nextPath = (strTotalPath + '/' +  item)
				confPath = (nextPath + '/conf/server.xml')
				intItemPort = self.fileUtil.getXMLTagElementValue(confPath, 'Connector', 'port', 0)
				listTomcatPort.append(intItemPort)
				listTomcatPath.append(nextPath)

			dictTomcatMsg['tomcatName'] = listTomcatName
			dictTomcatMsg['tomcatPort'] = listTomcatPort
			dictTomcatMsg['tomcatPath'] = listTomcatPath
			print(dictTomcatMsg)
			
			return dictTomcatMsg

这个tomcat检测模块中，可以比较自动识别tomcat安装个数及相对应的端口，操作xml配置文件的方法在fileUtil.py这个模块类中

#### redisCheck.py

redis检测模块
代码如下

	import os
	from monitorbin.util.process import ProcessCL

	#author: cg错过
	#time: 2017-09-30

	class RedisOperate:

		#redis检测模块
		
		def __init__(self, strRedisPath, intDateMin, fileUtilObj):

			#strRedisPath: redis的安装文件目录
			#intDateMin: 当前运行脚本的分钟数
			#fileUtilObj: FileUtil的对象(脚本从运行到结束都只有这一个FileUtil对象)

			self.fileUtil = fileUtilObj
			self.intDateMin = intDateMin
			self.strRedisPath = strRedisPath
			intCheckResult = self.fileUtil.checkFileExists(self.strRedisPath)
			if(intCheckResult == 1):
				self.checkRedis()
			else:
				self.fileUtil.writerContent("配置的redis路径不存在", 'runErr')

		def checkRedis(self):

			#每个小时检测一遍，不做操作
			#其他时候，当检测到未运行时，脚本尝试自启一次

			strRedisStatus = self.getRedisStatus()

			if(self.intDateMin == 30):
				self.checkRedisStatus(strRedisStatus)
			else:
				intMark = self.checkRedisStatus(strRedisStatus, 'Second')
				if(intMark == -1):
					self.tryStartRedis(self.strRedisPath)


		def getRedisStatus(self):

			#获取进程中的redis

			redisStatusCL = "ps -ef | grep redis"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(redisStatusCL)
			strRedisStatus = dictResult.get('stdout')
			return strRedisStatus


		def checkRedisStatus(self, strRedisStatus, strFileMark='Hour'):

			#判断redis是否运行

			intMark = -1
			strRedis = "redis-server"

			if(strRedisStatus.find(strRedis) != -1):
				#print("redis在运行")
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent("redis在运行")
				intMark = 1
			else:
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent("redis未运行")
				else:
					self.fileUtil.writerContent("redis未运行", 'Second')
				#print("redis未运行")
			return intMark


		def tryStartRedis(self, strRedisPath):

			#脚本启动redis

			intMark = -1
			#print("脚本尝试将其启动....")
			#print(strRedisPath)
			self.fileUtil.writerContent("脚本尝试将其启动....", 'Second')
			strStartRedisCL = strRedisPath + "/src/./redis-server"
			processCL = ProcessCL()
			dictResult = processCL.getContinueResultAndProcess(strStartRedisCL)
			strOut = dictResult.get('stdout')
			strErr = dictResult.get('stderr')
			if(strOut.find('redis.io') != -1):
				self.fileUtil.writerContent("redis已被脚本启动", 'Second')
				#print("redis已被脚本启动成功")
				intMark = 1
			else:
				#print("脚本启动redis未成功，请手动启动")
				self.fileUtil.writerContent("脚本启动redis未成功，请手动启动", 'Second')
				self.fileUtil.writerErr(strErr, 'Second')
				#print(strErr)
			return intMark
			
#### nginxCheck.py
nginx检测模块
代码如下

	from monitorbin.util.process import ProcessCL

	#author: cg错过
	#time: 2017-09-30

	class NginxOperate:

		#nginx检测模块

		def __init__(self, strNginxPath, intDateMin, fileUtilObj):
			
			#strNginxPath: nginx的安装目录
			#intDateMin: 当前运行脚本的分钟数
			#fileUtilObj: FileUtil的对象(脚本从运行到结束都只有这一个FileUtil对象)
			
			self.fileUtil = fileUtilObj
			self.intDateMin = intDateMin
			self.strNginxPath = strNginxPath
			intCheckResult = self.fileUtil.checkFileExists(self.strNginxPath)
			if(intCheckResult == 1):
				self.checkNginx()
			else:
				self.fileUtil.writerContent("配置的nginx路径不存在", 'runErr')

		def checkNginx(self):

			#每个小时检测一遍，不做操作
			#其他时候，当检测到未运行时，脚本尝试自启一次

			strNginxStatus = self.getNginxStatus()

			if(self.intDateMin == 30):
				self.checkNginxStatus(strNginxStatus)
			else:
				 intMark = self.checkNginxStatus(strNginxStatus, 'Second')
				 if(intMark == -1):
					 self.tryStartNginx(self.strNginxPath)



		def getNginxStatus(self):
			
			#获取进程中的nginx
			
			nginxStatusCL = "ps -ef | grep nginx"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(nginxStatusCL)
			strNginxStatus = dictResult.get('stdout')
			return strNginxStatus


		def checkNginxStatus(self, strNginxStatus, strFileMark='Hour'):

			#判断nginx是否运行

			intMark = -1
			strNginx = "nginx:"
			
			if(strNginxStatus.find(strNginx) != -1):
				#print("nginx在运行")
				intMark = 1
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent("nginx在运行")
			else:
				if(strFileMark=='Hour'):
					self.fileUtil.writerContent("nginx未运行")
				else:
					self.fileUtil.writerContent("nginx未运行", 'Second')
				#print("nginx未运行")
			return intMark


		def tryStartNginx(self, strNginxPath):

			#脚本启动nginx

			intMark = -1
			self.fileUtil.writerContent("脚本尝试将其启动....", 'Second')
			strStartNginxCL = strNginxPath + "/sbin/./nginx"
			processCL = ProcessCL()
			dictResult = processCL.getResultAndProcess(strStartNginxCL)
			strErr = dictResult.get('stderr')
			if(strErr == ''):
				#print("nginx已被脚本启动")
				self.fileUtil.writerContent("nginx已被脚本启动", 'Second')
				intMark = 1
			else:
				#print("脚本启动nginx未成功，请手动启动")
				self.fileUtil.writerContent("脚本启动nginx未成功，请手动启动", 'Second')
				self.fileUtil.writerErr(strErr, 'Second')
				#print(strErr)
			return intMark
		 
nginx和redis这两个检测模块代码比较类似

### monitor.conf
配置文件内容字段如下

	[ProjectConfigure]
	tomcatpath : tomcat安装目录的上一级目录，如果有多个tomcat,则为多个tomcat安装目录的上一级。需要这多个tomcat在同一目录下
	nginxpath : nginx安装目录
	redispath : redis安装目录

	[UseConfigure]
	servername : 服务器别称
	username : 发送邮件用户名

	[LogConfigure]
	logpath : 存放日志文件的路径名,例如'logpath :logs'即将在本脚本根目录下创建logs文件夹来存放日志

	[EmailConfigure]
	smtp_server : smtp服务地址,例如'smtp.qq.com'
	email_sendaddr : 发送邮件的邮箱账户
	email_sendpasswd : 发送邮件的账户授权码

	[ToEmail]
	//在此处填写接受邮件的邮箱地址，可以为多个，例如如下
	1732821152@qq.com

目前脚本简单检测这三个项目的功能已经实现，后期只需要在进程模块包中进行添加模块，同是配置文件也需要更改
后面再写，慢慢维护。项目代码已经提交至github仓库中，[这是里链接][]

[这篇文章]:http://www.firefoxbug.com/index.php/archives/2419/
[官方文档]:https://docs.python.org/2/library/subprocess.html
[中文文档]:https://www.rddoc.com/doc/Python/3.6.0/zh/library/subprocess/?highlight=subprocess
[这是里链接]:https://github.com/cgstudios/createUtil-daily-me/tree/master/python/automatic_monitor/automatic_monitor-base
