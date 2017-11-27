---
layout: post
title: Jenkins+Sonarqube 做Android代码质量检查
categories: [Jenkins, Sonarqube, Android]
description: Jenkins+Sonarqube 做Android代码质量检查
keywords: Jenkins, Sonarqube, Android
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
  3. Enter an item name 里输入项目名称，如 **代码检查-安卓代码**
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
	sonar.projectKey=qmzb-android-1
	sonar.projectName=qmzb-android
	sonar.projectVersion=1.0-${BUILD_NUMBER}
	sonar.sources=app/src/main/java
	sonar.binaries=app/build/intermediates/classes/  
	sonar.java.binaries=app/build/intermediates/classes/
	sonar.language=java
	sonar.sourceEncoding=UTF-8
	
	sonar.exclusions=app/src/main/java/com/gele/mryan/quanminzhibo/a_app/authpack.java
	#sonar.profile= SonarAndroidLint 
	```

  12. 点击**保存**按钮
  
  


# Step4. 立即构建任务

# Step5. 在SonarQube中查看结果


