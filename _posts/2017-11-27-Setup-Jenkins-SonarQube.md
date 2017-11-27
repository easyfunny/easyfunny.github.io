---
layout: post
title: Jenkins+Sonarqube 做java代码质量检查
categories: [Jenkins, Sonarqube]
description: Jenkins+Sonarqube 做java代码质量检查
keywords: Jenkins, Sonarqube
---

# Step1. 安装SonarQube Scanner for Jenkins
  1. 进入Jenkins->插件管理->可选插件
  2. 右侧Filter输入sonar
  3. 安装SonarQube Scanner for Jenkins
  4. 重启Jenkins
 
# Step2. 配置SonarQube
  1. 进入Jenkins->配置
  2. 滚动到 **SonarQube servers**，点击**Add SonarQube**按钮
  3. 添加一个Server，如下：
  	* Name输入Sonar
  	* Server URL输入SonarQube的访问地址，如*http://192.168.1.240:9000*
	* Server version 为 *5.3 or higher*
	* Server authentication token 填写 SonarQube中自动生成的token
  4. 点击保存

# Step3. 新建一个Jenkins任务
  1. 进入Jenkins
  2. 点击“**新建**”按钮
  3. Enter an item name 里输入项目名称，如 **代码检查-后端代码**
  4. 选择**构建一个自由风格的软件项目**
  5. 点击**OK**按钮
  6. **源码管理**章节选择**Git**
  7. **Repository URL**输入git地址
  8. **Credentials**选择或者新建一个访问凭证
  9. **构建触发器**选择**Poll SCM**，**日程表**输入Cron表达式，如
  		`H/5 * * * *`
  		是每5分钟一次
  10. **构建**选择**Execute SonarQube Scanner**
  11. **Analysis properties**输入以下脚本
	
	```
	# must be unique in a given SonarQube instance
	sonar.projectKey=qmzb-server-1
	# this is the name displayed in the SonarQube UI
	sonar.projectName= qmzb-server
	sonar.projectVersion=1.0-${BUILD_NUMBER}
	 
	# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
	# Since SonarQube 4.2, this property is optional if sonar.modules is set. 
	# If not set, SonarQube starts looking for source code from the directory containing 
	# the sonar-project.properties file.
	
	
	sonar.modules= java-module,xml-module
	sonar.java.binaries= APIUtils/libs/
	sonar.java.libraries= APIUtils/libs/
	
	java-module.sonar.projectName=Java Module  
	java-module.sonar.language=java
	java-module.sonar.sources=APIController/src/main/java/,APIEntity/src/main/java/,APIService/src/main/java/,APIServiceImpl/src/main/java/,APIUtils/src/main/java/
	java-module.sonar.projectBaseDir=./
	sonar.exclusions=**/com/alibaba/**
	
	xml-module.sonar.projectName=Xml Module  
	xml-module.sonar.language=xml
	xml-module.sonar.sources=APIServiceImpl/src/main/resources/mapper/
	xml-module.sonar.projectBaseDir=./
	# Encoding of the source code. Default is default system encoding
	sonar.sourceEncoding=UTF-8
	```

  12. 点击**保存**按钮
  
  


# Step4. 立即构建任务

# Step5. 在SonarQube中查看结果


