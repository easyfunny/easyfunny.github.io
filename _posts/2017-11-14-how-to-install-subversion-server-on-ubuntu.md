
---
layout: post
title: 如何在Ubuntu 16.04和14.04 LTS上安装Subversion Server
categories: [ubuntu, svn, Subversion Server]
description: 如何在Ubuntu 16.04和14.04 LTS上安装Subversion Server
keywords: ubuntu, OpenLDAP, phpLDAPadmin
---
# 如何在Ubuntu 16.04和14.04 LTS上安装Subversion Server

> 如何在Ubuntu 16.04和14.04 LTS上安装Subversion Server。

## 介绍
Subversion是一个开源版本控制系统。它可以帮助您跟踪文件和文件夹的集合。任何时候更改，添加或删除您使用Subversion管理的文件或文件夹，都会将这些更改提交到Subversion存储库，这会在存储库中创建一个反映这些更改的新版本。你可以随时回去，看看并获得以前版本的内容。 本文将帮助您在Ubuntu 16.04和14.04 LTS系统上逐步设置Subversion（svn）服务器。

## 步骤

### 1. 安装Apache
首先，您需要安装Apache Web服务器以使用http网址访问svn服务器。如果您的系统上已有Apache Web服务器，请跳过此步骤。

```
sudo apt-get update
sudo apt-get install apache2
```

#### 1.1 打开SSL
```
sudo a2enmod ssl
sudo nano /etc/apache2/ports.conf
sudo a2ensite default-ssl
sudo a2dissite 000-default
```

### 2. 安装Subversion
使用以下命令安装Subversion软件包和依赖关系。还在您的系统上安装Apache libapache2-mod-svn软件包的svn模块。

```
sudo apt-get install subversion libapache2-mod-svn libapache2-svn libsvn-dev
sudo a2enmod dav
sudo a2enmod dav_svn
sudo service apache2 restart
```

### 3.创建第一个SVN存储库
使用下面的命令来创建一个名为**myrepo**你的第一个svn存储库。

```
sudo mkdir -p /home/mduser/svndata
sudo svnadmin create /home/mduser/svndata/myrepo
sudo chown -R www-data:www-data /home/mduser/svndata
sudo chmod -R 775 /home/mduser/svndata
```

### 4.为Subversion创建用户
现在，在创建文件**/etc/apache2/dav_svn.passwd**第一SVN用户。这些用户将使用svn存储库的认证进行检出，提交进程。

```
sudo htpasswd -cm /etc/apache2/dav_svn.passwd admin
```

要创建其他用户，请使用以下命令。

```
sudo htpasswd -m /etc/apache2/dav_svn.passwd user1
sudo htpasswd -m /etc/apache2/dav_svn.passwd user2
```

### 5. 配置Apache for Subversion
Subversion的Apache模块包创建一个配置文件**/etc/apache2/mods-enabled/dav_svn.conf**。你只需要对其进行必要的更改。

```
Alias /svn /home/mduser/svndata
<Location /svn>
	DAV svn
	SVNParentPath /home/mduser/svndata
	AuthType Basic
	AuthName "My FUCKING Subversion Repository"
	AuthUserFile /etc/apache2/dav_svn.passwd
	Require valid-user
	SSLRequireSSL
</Location>
```

### 6. 在浏览器中访问存储库
使用http网址在浏览器中访问您的存储库。它将提示进行身份验证。使用在步骤5中创建的登录凭据。使用系统主机名，域名或IP地址更改example.com。

```
http://example.com/svn/myrepo/
```