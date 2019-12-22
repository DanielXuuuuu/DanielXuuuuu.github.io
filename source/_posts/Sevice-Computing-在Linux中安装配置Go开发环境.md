---
title: Sevice Computing - 在Linux中安装配置Go开发环境
date: 2019-09-10 19:43:41
categories: 
	- 服务计算
tags: 
	- Go语言	
---

> 本文主要记录了我在Linux中配置Go开发环境过程中遇到的一些问题和奇怪的坑。由于CentOS虚拟机较卡，后来换用了Ubuntu。

{% asset_img go.jpg %}

<!--more-->

[完整过程指导](https://pmlpml.github.io/ServiceComputingOnCloud/ex-install-go)

##### CentOS

###### yum不可使用

> 在安装好CentOS和Chrome后，yum突然不可用了，更换了好几次源还是不行。

经过上网查询，这可能是安装了Chrome导致的，参考了别人的解决方法后才解决：

```/
rm -rf /etc/yum.repos.d/google-chrome.repo
yum clean all
```

###### Golang的安装

> 在使用命令`sudo yum install golang`时提示没有可用软件包，并且已经执行过`yum update`了。

这代表在linux系统yum源中已经没有对应的安装包了，这时我们需要安装EPEL(Extra Packages for Enterprise Linxu，企业版Linux额外包)：

```/
yum install -y epel-release
```

###### Git安装

> 通常情况下通过使用命令`sudo yum install git`来安装的git版本都较低，我在使用VSCode时，提示我需要升级。于是我参考[此博客](https://www.cnblogs.com/kevingrace/p/8252517.html)进行了升级，但升级过程中，在执行`make prefix=/usr/local/git all`进行编译时，出现了以下两个问题：

+ 找不到openssl/ssl.h文件。解决方法：`yum install openssl-devel`
+ 找不到curl/curl.h。解决方法：`yum -y install curl-devel`

###### VSCode中安装Go相关插件

> 在使用VSCode打开.go文件后，按照提示下载插件，最后提示有一个安装包安装失败`Installing github.com/ramya-rao-a/go-outline FAILED`。然后按照指导页面使用`go get -d github.com/golang/tools`将源代码下载到本地后安装还是不行。

解决方法：通过手动安装

```/
go get -u github.com/ramya-rao-a/go-outline
go install github.com/ramya-rao-a/go-outline
```

##### Ubuntu

###### 环境变量

> 在设置GOPATH环境变量时，按照教程先后执行`export GOPATH=/home/daniel/go`和`export PATH=$PATH:$GOPATH/bin`后，仅仅只在当前终端生效。关闭后重新打开终端，上述配置失效。

解决方法：

```/
# 通过修改.bashrc文件
vim ~/.bashrc
# 在最后面加上下面两句：
export GOPATH=/home/daniel/go # 这是我的设置
export PATH=$PATH:$GOPATH/bin
# 保存退出后，执行以下命令
source ~/.bashrc 
```

上述解决方法对当前用户永久有效，若是想要对所有用户永久有效，就要修改`etx/profile`文件，修改方式相同。