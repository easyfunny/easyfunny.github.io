---
title: "软件开发管理环境配置-中小团队"
description: "Setup gitlab server and redmine for small team"
category: "development"
tags: [Project Management, Configuration Management, git, ]
---

# 一.概述
=======
- Jenkins 安装
- SonarQube 安装
- LDAP集成


# 二.具体安装配置
=============

## 2.1 Jenkins 安装
-------------

1. 安装JRE

  `sudo apt-get install -y openjdk-8-jdk`

2. 下载 Jenkins 2.73.3（LTS）, 并安装

	```
	wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
	mkdir jenkinswork
	```
	
	编写startjen.sh
	
	```
	export JENKINS_HOME=/home/videa/jenkinswork
	nohup java -jar /home/videa/jenkins.war  --httpPort=18080> /home/videa/jenkins.log &
	```
	
	`chmod +x startjen.sh`
	
	设置cron为开机启动
	
	`crontab -e`
	
	`@reboot /home/videa/startjen.sh`
	

    - 缺省端口：18080

## 2.2 SonarQube 安装
----------------
1. 下载 SonarQube 6.7 (LTS *)，

	```wget wget https://sonarsource.bintray.com/Distribution/sonarqube/sonarqube-6.7.zip
	sudo apt-get install -y zip
	unzip sonarqube-6.7.zip
	```
	
	- If you're running on Linux, you must ensure that:
		- vm.max_map_count is greater or equals to 262144
		- fs.file-max is greater or equals to 65536
		- the user running SonarQube can open at least 65536 file descriptors

	You can see the values with the following commands :
	
	```
	sysctl vm.max_map_count
	sysctl fs.file-max
	ulimit -n
	```
	
	You can set them dynamically for the current session by running  the following commands as **root**:
	
	```
	sudo sysctl -w vm.max_map_count=262144
	sudo sysctl -w fs.file-max=65536
	sudo ulimit -n 65536
	```


2. 创建数据库和用户:
       
   ```
	CREATE DATABASE sonar6 CHARACTER SET utf8 COLLATE utf8_general_ci; 
	CREATE USER 'sonar6' IDENTIFIED BY 'sonar6';
	GRANT ALL ON sonar6.* TO 'sonar6'@'%' IDENTIFIED BY 'sonar6';
	GRANT ALL ON sonar6.* TO 'sonar6'@'localhost' IDENTIFIED BY 'sonar6';
	FLUSH PRIVILEGES;
	```

4. 配置 sonarqube-6.7/conf/sonar.properties  使用 mysql数据库

	```
   sonar.jdbc.username=sonar6
   sonar.jdbc.password=sonar6
   sonar.jdbc.url=jdbc:mysql://mini.md:3306/sonar6?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
	```
	
5. 运行

   `./sonarqube-6.7/bin/linux-x86-64/sonar.sh start`
   
   
## 2.3 Redmine 配置
----------------

> http://www.redmine.org/projects/redmine/wiki/HowTo_Install_Redmine_on_Ubuntu_step_by_step

1. 安装 apache2 mysql

	`sudo apt-get install -y apache2 libapache2-mod-passenger mysql-server mysql-client`
       修改root密码， 并创建redmine用户（mysql用户） -->The user is for redmine web application.

2. Deployment redmine to /opt/redmine3

       记住redmine admin用户密码

3. Install unicorn and nginx to power redmine （也可使用Passenger等其他Server）

       运行环境： http://MiniServer:3000 , --> 具体配置请参见Unicorn和Nginx集成文档

4. 配置LDAP， 利用admin用户登录Redmine的页面，进行配置即可


## 2.4 LDAP-Account-Manager配置
-----------------------------------

为了方便团队成员添加用户（例如新员工加入等）， 使用ldap-account-manager管理ApacheDS的员工帐号

1. 安装ldap-account-manager,lam to manage LDAP accounts
   - install php
   - download lam tar.bz 包， 解压缩到/opt/lam

2. 在 nginx 添加一个Site，监听在 3300端口
   - 直接在lam页面上添加一个LDAP Profile，在页面上配置即可


## 2.5 Self LDAP Change Password
-----------------------------------

为了方便员工自己修改密码，部署LTB - Self Service Prassword

1. 选择LTB - Self Service Prassword，  部署到Nginx， 和LAM 使用同一个文件， /passwd Location

        location /lam {
                index index.html;
                alias /opt/lam;
                autoindex off;

                location ~ \.php$ {
                        fastcgi_split_path_info ^(.+\.php)(/.+)$;
                        fastcgi_pass unix:/var/run/php5-fpm.sock;
                        fastcgi_index index.php;
                        include fastcgi_params;
                }

                location ~ /lam/(tmp/internal|sess|config|lib|help|locale) {
                        deny all;
                        return 403;
                }
        }

        location /passwd {
                index index.php;
                alias /opt/self_ldap_passwd;
                autoindex off;
                location ~ \.php$ {
                        fastcgi_split_path_info ^(.+\.php)(/.+)$;
                        fastcgi_pass unix:/var/run/php5-fpm.sock;
                        fastcgi_index index.php;
                        include fastcgi_params;
                }

        }

2. 编辑/opt/self_ldap_passwd/conf/confic.inc.php 中的 LDAP 连接信息

       $ldap_url = "ldap://ServerLDAP:10389";
       $ldap_binddn = "uid=useradmin,ou=system";
       $ldap_bindpw = "xxxxxx";
       $ldap_base = "dc=hello,dc=com";
       $ldap_login_attribute = "uid";


## 2.6 FTP配置
---------------

项目中的大文件签入Git会造成git速度下降， 所以大的二进制文件使用FTP管理

1. 下载FileZilla Server, 安装
2. 配置FTP Server

     可以下载FileZilla Server Inferface（GUI Tool）来配置FileZilla FTP Server
     连接信息： 127.0.0.1  端口：14147， 密码：xxxxx （自己指定）
	 
     [*Troubleshooting*]
	 
     使用FileZilla Server 作为FTP服务器， 刚开始login和list非常慢，
     我的尝试解决办法：
	 
     1. 首先使用服务器管理器的“windows高级安全防火墙”添加端口， 没有解决问题
     2. 使用控制面板中的windows防火墙吧“FileZilla Server”和“FileZilla Admin Server 都加进去， 然后全部启用，就好了， 记录一下， 以后备查。

ftp的根目录 d:\ftp_home

     权限配置为： 所有人匿名可读可上传， 可匿名上传到临时目录， 由文档管理人员整理到正式目录
     临时目录：uploadings,   -->供用户零时上传的目录，
     正式目录：projects， dept_documents， tools， softwares


## 2.7 其他
--------

其他可以配置， Code Review  （To be done)

