
---
layout: post
title: 如何在Ubuntu 16.04上安装和配置OpenLDAP和phpLDAPadmin
categories: [ubuntu, OpenLDAP, phpLDAPadmin]
description: 在本指南中，我们将讨论如何在Ubuntu 16.04上安装和配置OpenLDAP服务器。然后，我们将安装phpLDAPadmin，一个用于查看和操作LDAP信息的Web界面。我们将使用免费和自动化证书提供商Let's Encrypt来保护Web界面和LDAP服务的SSL证书。
keywords: ubuntu, OpenLDAP, phpLDAPadmin
---
# 如何在Ubuntu 16.04上安装和配置OpenLDAP和phpLDAPadmin

> 在本指南中，我们将讨论如何在Ubuntu 16.04上安装和配置OpenLDAP服务器。然后，我们将安装phpLDAPadmin，一个用于查看和操作LDAP信息的Web界面。我们将使用免费和自动化证书提供商Let's Encrypt来保护Web界面和LDAP服务的SSL证书。

##介绍
轻量级目录访问协议（LDAP）是用于通过网络管理和访问分层目录信息的标准协议。 它可以用于存储任何类型的信息，尽管它通常用作集中式身份验证系统或公司电子邮件和电话目录。

在本指南中，我们将讨论如何在Ubuntu 16.04上安装和配置OpenLDAP服务器。 然后，我们将安装phpLDAPadmin，一个用于查看和操作LDAP信息的Web界面。 我们将使用免费和自动化证书提供商Let's Encrypt来保护Web界面和LDAP服务的SSL证书。

##先决条件
在开始本教程之前，您应该使用Apache和PHP设置Ubuntu 16.04服务器。 您可以按照我们的教程如何在Ubuntu 16.04上安装Linux，Apache，MySQL，PHP（LAMP） ，跳过第2步，因为我们不需要MySQL数据库服务器。

另外，由于我们将在Web界面中输入密码，所以我们应该使用SSL加密保护Apache。 阅读如何保护Apache，让我们在Ubuntu 16.04上加密下载和配置免费的SSL证书。 您将需要一个域名来完成此步骤。 我们将使用这些相同的证书来提供安全的LDAP连接。

**注意**：我们的加密教程假设您的服务器可以访问公共互联网。 如果不是这种情况，您将不得不使用不同的证书提供商或者组织自己的证书颁发机构。 无论哪种方式，您应该能够以最少的更改完成教程，主要是关于证书的路径或文件名。

###第1步 - 安装和配置LDAP服务器
我们的第一步是安装LDAP服务器和一些相关的实用程序。 幸运的是，我们需要的软件包都可以在Ubuntu的默认存储库中使用。

登录到您的服务器。 由于这是我们第一次在本次会议中使用apt-get ，所以我们将刷新本地软件包索引，然后安装我们想要的软件包：

```
sudo apt-get update
sudo apt-get install slapd ldap-utils
```

在安装过程中，将要求您选择并确认LDAP的管理员密码。 您可以在这里输入任何内容，因为您将有机会在短时间内进行更新。

即使我们刚刚安装了这个软件包，我们也要进行重新配置。 slapd软件包有能力提出很多重要的配置问题，但默认情况下，它们将在安装过程中跳过。 通过告诉我们的系统重新配置包，我们可以访问所有提示：

```
sudo dpkg-reconfigure slapd
```

在这个过程中有很多新的问题需要回答。 我们将接受大部分违约。 我们来回答一下问题：

* 省略了OpenLDAP服务器配置？ 没有
* DNS域名？
	* 此选项将确定目录路径的基本结构。 阅读消息以了解这将如何实现。 即使您不拥有实际的网域，您也可以选择所需的任何值。 但是，本教程假设您具有适当的服务器域名，因此您应该使用它。 我们将在整个教程中使用**example.com** 。
* 机构名称？
	* 对于本指南，我们将使用示例作为我们组织的名称。 你可以选择任何你觉得合适的东西。
* 管理员密码？ 输入两次安全密码
* 数据库后端？ MDB
* 清除slapd时删除数据库？ 没有
* 移动旧数据库？ 是
* 允许LDAPv2协议？ 没有

此时，您的LDAP服务器已配置并运行。 打开防火墙上的LDAP端口，以便外部客户端可以连接：

```
sudo ufw allow ldap
```

我们来测试我们与ldapwhoami的LDAP连接，该连接应该返回我们连接的用户名：

````
ldapwhoami -H ldap:// -x
````

````
Output
anonymous
````

anonymous是我们anonymous的结果，因为我们运行ldapwhoami而不登录到LDAP服务器。 这意味着服务器正在运行并应答查询。 接下来，我们将设置一个Web界面来管理LDAP数据。

### 第2步 - 安装和配置phpLDAPadmin Web界面
尽管很可能通过命令行管理LDAP，但大多数用户会发现使用Web界面更为容易。 我们将安装phpLDAPadmin，这是一个提供此功能的PHP应用程序。

Ubuntu存储库包含一个phpLDAPadmin软件包。 您可以使用apt-get安装它：

```
sudo apt-get install phpldapadmin
```

这将安装应用程序，启用必要的Apache配置，并重新加载Apache。

Web服务器现在配置为提供应用程序，但我们需要进行一些其他更改。 我们需要将phpLDAPadmin配置为使用我们的域，而不是自动填充LDAP登录信息。

首先在文本编辑器中打开具有root权限的主配置文件：

```
sudo nano /etc/phpldapadmin/config.php
```

寻找以**$servers->setValue('server','name'**开头的行，在**nano**您可以通过键入**CTRL-W** ，然后输入字符串然后**ENTER**来搜索字符串，您的光标将被放置在正确的线。

该行是您的LDAP服务器的显示名称，Web界面用于有关服务器的标题和消息。 在这里选择适当的选择：


```
$servers->setValue('server','name','Example LDAP');
```

接下来，向下移动到**$servers->setValue('server','base'**行，该配置告诉phpLDAPadmin LDAP层次结构的根目录，这是基于我们在重新配置slapd包时输入的值。我们的示例我们选择了example.com ，我们需要将每个域组件（不是一个点）放入dc= notation中将其转换为LDAP语法：

```
$servers->setValue('server','base', array('dc=example,dc=com'));
```

现在找到登录bind_id配置行，并bind_id开头注释#

```
#$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

此选项预先填充Web界面中的管理员登录详细信息。 这是我们不能共享的信息，如果我们的phpLDAPadmin页面是可公开访问的。

我们需要调整的最后一件事是控制一些phpLDAPadmin警告消息的可见性的设置。 默认情况下，应用程序将显示相当多的关于模板文件的警告消息。 这些对我们目前使用的软件没有影响。 我们可以通过搜索hide_template_warning参数来隐藏它们，取消注释包含它的行，并将其设置为**true** ：

```
$config->custom->appearance['hide_template_warning'] = true;
```

这是我们需要调整的最后一件事。 保存并关闭文件以完成。 我们不需要重新启动任何更改才能生效。

接下来我们将登录到phpLDAPadmin。

###第3步 - 登录到phpLDAPadmin Web界面
将必要的配置更改为phpLDAPadmin后，我们现在可以开始使用它了。 浏览您的网页浏览器中的应用程序。 请务必将您的域替换为以下突出显示的区域：

`https://example.com/phpldapadmin`

phpLDAPadmin着陆页将加载。 点击页面左侧菜单中的登录链接。 将提供登录表单：

phpLDAPadmin登录页面

登录DN是您将要使用的用户名。 它包含帐户名称作为cn=部分，您为服务器选择的域名分为dc=部分，如上述步骤所述。 我们在安装过程中设置的默认管理员帐户称为admin ，因此在我们的示例中，我们将键入以下内容：

`cn=admin,dc=example,dc=com`

为您的域输入适当的字符串后，键入您在配置期间创建的管理员密码，然后单击验证按钮。

您将被带到主界面：

phpLDAPadmin主页

此时，您将登录到phpLDAPadmin界面。 您可以添加用户，组织单位，组和关系。

LDAP如何构建数据和目录层次结构是灵活的。 您可以创建任何类型的结构，并创建如何交互的规则。

由于Ubuntu 16.04上的过程与以前的版本相同，因此您可以按照Ubuntu 12.04的LDAP安装文章的 “ 添加组织单位，组和用户”部分中的步骤进行操作。

这些步骤将在此安装phpLDAPadmin上运行良好，因此请遵循以下操作，以了解如何使用界面并学习如何构建数据。

现在我们已经登录并熟悉了Web界面，让我们花点时间为LDAP服务器提供更多的安全性。

## 结论
在本教程中，我们安装并配置了OpenLDAP slapd服务器和LDAP Web界面phpLDAPadmin。

我们设置的系统非常灵活，您可以根据需要设计自己的组织架构并管理资源组。 有关管理LDAP的更多信息，包括更多的命令行工具和技术，请阅读我们的教程如何使用OpenLDAP实用程序管理和使用LDAP服务器。