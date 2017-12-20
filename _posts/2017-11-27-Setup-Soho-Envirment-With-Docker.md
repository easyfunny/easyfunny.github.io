---
layout: post
title: Docker建立内部办公集成环境
categories: [Docker]
description: Docker建立内部办公集成环境
keywords: Docker, Jenkins, SonarQube, Redmine, Mysql, nextCloud, ApacheDS
---

# 使用Docker建立内部办公集成环境
**2017-11-30**
> 主要介绍使用Docker建立内部办公集成环境，基于CentOS 7系统，主要是用了Mysql作为数据库，ApacheDS作为单点登录，Redmine作为项目管理软件，Jenkins做CI，SonarQube做代码检查。
> **未来会增加GitLab、TestLink等。**

## 安装Docker
使用以下语句安装Docker

```
sudo yum install docker
```

## 下载Docker镜像

```
sudo docker pull mysql:5.6
sudo docker pull openmicroscopy/apacheds
sudo docker pull sameersbn/redmine
sudo docker pull jenkins/jenkins
sudo docker pull sonarqube
sudo docker pull nextcloud
```


## 定义并运行容器
> **注意**，--link 参数，表明不同容器间的调用关系。不写会导致无法通信。
>
> -v 属性用来控制存储空间。
>
> 修改下面的邮箱地址、邮箱密码。
>

```
sudo docker run -d  --name mysql.qmzb.local -v mysqldata:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 mysql:5.6 
sleep 10

sudo docker run -d  --name apacheds.qmzb.local -p 10389:10389 openmicroscopy/apacheds
sleep 10

sudo docker run -d  --name redmine.qmzb.local -p 8080:80 -e DOCKER_HOST=192.168.2.239 -e DB_HOST=mysql.qmzb.local -e DB_TYPE=mysql -e DB_NAME=redmine -e DB_USER=redmine -e DB_PASS=redmine -e TZ=Asia/Kolkata -e REDMINE_PORT=8080 -e SMTP_ENABLED=true -e SMTP_DOMAIN=www.126.com -e SMTP_HOST=smtp.126.com -e SMTP_PORT=25 -e SMTP_USER=邮箱地址 -e SMTP_PASS=邮箱密码 -e SMTP_STARTTLS=false -e SMTP_AUTHENTICATION=:login -e IMAP_ENABLED=false -e IMAP_USER=mailer@example.com -e IMAP_PASS=password -e IMAP_HOST=imap.gmail.com -e IMAP_PORT=993 -e IMAP_SSL=true -e IMAP_INTERVAL=30 --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local sameersbn/redmine

sudo docker run -d  --name sonarqube.qmzb.local -p 9000:9000 -e SONARQUBE_JDBC_USERNAME=sonar -e SONARQUBE_JDBC_PASSWORD=sonar -e DB_ADAPTER=mysql -e SONARQUBE_JDBC_URL="jdbc:mysql://mysql.qmzb.local:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance" -v sonarqubedata:/opt/sonarqube/data  --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local --link redmine.qmzb.local:redmine.qmzb.local sonarqube

sudo docker run -d  --name nextcloud.qmzb.local -v nextcloud:/var/www/html -v nextcloudapps:/var/www/html/custom_apps -v nextcloudconfig:/var/www/html/config -v nextclouddata:/var/www/html/data -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=nextcloud -e MYSQL_HOST=mysql.qmzb.local -p 8000:80  --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local nextcloud

sudo docker run -d -p 8100:80 --name testlink.qmzb.local -v testlink:/bitnami -e MARIADB_USER=root -e MARIADB_PASSWORD=123456 -e MARIADB_HOST=mysql.qmzb.local -e TESTLINK_USERNAME=admin123 -e TESTLINK_PASSWORD=123456 --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local --link redmine.qmzb.local:redmine.qmzb.local bitnami/testlink

sudo docker run -d --hostname gitlab.qmzb.local -p 80:80 --publish 8222:22 --name gitlab.qmzb.local --restart always --volume gitlabconfig:/etc/gitlab:Z --volume gitlablogs:/var/log/gitlab:Z --volume gitlabdata:/var/opt/gitlab:Z  --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local --link redmine.qmzb.local:redmine.qmzb.local gitlab/gitlab-ce

sudo docker run -d  --name jenkins.qmzb.local -p 8090:8080 -v jenkinsdata:/var/jenkins_home  --link mysql.qmzb.local:mysql.qmzb.local --link apacheds.qmzb.local:apacheds.qmzb.local --link sonarqube.qmzb.local:sonarqube.qmzb.local --link gitlab.qmzb.local:gitlab.qmzb.local jenkins/jenkins


```


## 执行数据库脚本

```
CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'sonar' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
FLUSH PRIVILEGES;

CREATE DATABASE redmine CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'redmine' IDENTIFIED BY 'redmine';
GRANT ALL ON redmine.* TO 'redmine'@'%' IDENTIFIED BY 'redmine';
GRANT ALL ON redmine.* TO 'redmine'@'localhost' IDENTIFIED BY 'redmine';
FLUSH PRIVILEGES;

CREATE DATABASE nextcloud CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'nextcloud' IDENTIFIED BY 'nextcloud';
GRANT ALL ON nextcloud.* TO 'nextcloud'@'%' IDENTIFIED BY 'nextcloud';
GRANT ALL ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY 'nextcloud';
FLUSH PRIVILEGES;

/* 不需要了 
CREATE DATABASE testlink CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'testlink' IDENTIFIED BY 'testlink';
GRANT ALL ON testlink.* TO 'testlink'@'%' IDENTIFIED BY 'testlink';
GRANT ALL ON testlink.* TO 'testlink'@'localhost' IDENTIFIED BY 'testlink';
FLUSH PRIVILEGES;
*/
```

## 停止容器

```
sudo docker stop jenkins.qmzb.local
sudo docker stop gitlab.qmzb.local
sudo docker stop testlink.qmzb.local
sudo docker stop redmine.qmzb.local
sudo docker stop sonarqube.qmzb.local
sudo docker stop nextcloud.qmzb.local
sudo docker stop mysql.qmzb.local
sudo docker stop apacheds.qmzb.local
```

## 启动容器

```
sudo docker start mysql.qmzb.local
sudo docker start apacheds.qmzb.local


sleep 5
sudo docker start redmine.qmzb.local
sudo docker start sonarqube.qmzb.local
sudo docker start nextcloud.qmzb.local
sudo docker start testlink.qmzb.local
sleep 5
sudo docker start gitlab.qmzb.local
sleep 15
sudo docker start jenkins.qmzb.local
```

## 数据存储位置，用于定期备份
有些已经起了别名，有些不可以用别名，为随机数。

```
/var/lib/docker/volumes/
```

## 配置gitlab，使用ldap认证

```
sudo docker exec -it gitlab.qmzb.local nano /etc/gitlab/gitlab.rb
```

```
gitlab_rails['ldap_enabled'] = true

gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
main: # 'main' is the GitLab 'provider ID' of this LDAP server
   label: 'ldap-team'
   host: 'apacheds.qmzb.local'
   port: 10389
   uid: 'uid'
   method: 'plain'
   bind_dn: 'uid=admin,ou=system'
   password: 'secret'
   active_directory: true
   allow_username_or_email_login: false
   base: 'ou=employees,dc=geekdana,dc=com'
EOS
```

## 已知问题
  1. 偶尔apacheds启动后自动关闭，原因不明。重复启动多次后可正常，建议apacheds多次执行启动脚本。
  2. 容器间通信，使用别名访问，不能以ip方式访问。

## 下一步
  1. 安装Nexus等。
  2. 配置ApacheDS的用户。
  3. 将Jenkins、SonarQube、 Redmine、nextCloud、GitLab、TestLink都连接到ApacheDS上以实现单点登录。
