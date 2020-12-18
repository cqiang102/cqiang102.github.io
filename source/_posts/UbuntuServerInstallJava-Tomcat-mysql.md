---
title: Ubuntu Server 安装 Java、Tomcat、mysql
date: 2019-6-27 17:11:26
categories: 笔记
tags: [linux]
type: posts
description: 记录一下 Ubuntu Server 安装 Java、Tomcat、mysql
---
### 安装Java

使用xftp把压缩包上传到Ubuntu 中，我这里使用的是JDK1.8 
> https://www.oracle.com/technetwork/java/javase/downloads/index.html

解压缩


```
tar -zxvf jdk-8u152-linux-x64.tar.gz
```

创建目录

```
mkdir -p /usr/local/java
```

移动解压好的安装包

    
```
mv jdk1.8.0_152/ /usr/local/java/
```

设置所有者

```
chown -R root:root /usr/local/java/
```
配置系统环境变量

```
vim /etc/environment

在第二行加入
export JAVA_HOME=/usr/local/java/jdk1.8.0_152
export JRE_HOME=/usr/local/java/jdk1.8.0_152/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
```
配置用户环境变量

```
/etc/profile

修改为如下

if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
    # The file bash.bashrc already sets the default PS1.
    # PS1='\h:\w\$ '
    if [ -f /etc/bash.bashrc ]; then
      . /etc/bash.bashrc
    fi
  else
    if [ "`id -u`" -eq 0 ]; then
      PS1='# '
    else
      PS1='$ '
    fi
  fi
fi
#加入的语句
export JAVA_HOME=/usr/local/java/jdk1.8.0_152
export JRE_HOME=/usr/local/java/jdk1.8.0_152/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH:$HOME/bin
#加入的语句

if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
    if [ -r $i ]; then
      . $i
    fi
  done
  unset i
fi
```

使环境变量生效

```
source /etc/profile
```
测试安装是否成功

```
java -version
```


### 安装Tomcat

我这里安装的是Tomcat 8.5.23
> https://tomcat.apache.org/

解压缩上传的压缩文件


```
tar -zxvf apache-tomcat-8.5.23.tar.gz
```
改成简短的文件名

```
mv apache-tomcat-8.5.23 tomcat
```
移动目录

```
mv tomcat/ /usr/local/
```
启动
执行tomcat/bin/目录下的`startup.sh`


### 安装MySql

更新数据源

```
apt-get update
```
安装


```
apt-get install mysql-server

#安装途中需要设置root密码
```


如遇到如下问题,则执行划线部分代码

> E: dpkg was interrupted, you must manually run '++dpkg --configure -a++' to correct the problem

配置允许远程连接

```
vim /etc/mysql/mysql.conf.d/mysqld.cnf

#注释掉如下行（前面加上#号）
bind-address = 127.0.0.1
```

重启Mysql服务

```
service mysql restart
```
授权 root 用户允许所有人连接

```
grant all privileges on *.* to 'root'@'%' identified by '你的 mysql root 账户密码';
```


如果出现如下情况，是由于密码安全级别太低

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

使用如下设置即可

set global validate_password_policy=0;  //设置密码安全级别
set global validate_password_length=1;  //设置密码长度限制
```
重启Mysql服务

```
service mysql restart
```
