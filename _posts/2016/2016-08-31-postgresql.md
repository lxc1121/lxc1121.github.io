---
layout: post
title: rsyslog+PostgreSQL
categories:
- system config
tags:
- rsyslog
- Postgresql
- database
---

## PostgreSQL 安装

	yum install postgresql postgresql-server

正常情况下，安装完成后，PostgreSQL服务器会自动在本机的5432端口开启。
初次安装后，默认生成一个名为postgres的数据库和一个名为postgres的数据库用户。

**这里需要注意的是，同时还生成了一个名为postgres的Linux系统用户。**

## 创建用户

通过命令su - postgres进入postgresql管理账户

	psql 			/*登录PostgreSQL控制台,相当于命令psql -U postgres -d postgres*/

psql 默认使用当前登录用户名登陆同名数据库，意思是如果当是root用户，那么psql=psql -U root -d root

	\password postgres 	/*为用户postgres设置密码,可以不*/
	create role rsyslog	/*创建用户rsyslog*/
	\password rsyslog 	/*为用户rsyslog设置密码，123456*/

## 创建数据库

	create database "Syslog" owner=rsyslog /**数据库名是大写字母必须要用双引号括起来**/
	grant all on database "Syslog" to rsyslog /**为用户rsyslog授权**/

数据库创建完成后\q退出，执行以下命令测试
	
	psql -U rsyslog -d Syslog	/*以用户rsyslog登录并连接到Syslog数据库*/

理论上能正常登陆数据库，\l命令查看数据库Syslog的授权情况

## 配置rsyslog 在数据库中的表项

* yum install rsyslog-pgsql 安装rsyslog的postgresql驱动模块
* 编辑文件/usr/share/doc/rsyslog-7.4.7/pgsql-createDB.sql**注释第一行**
* psql -U rsyslog -d Syslog &lt; /usr/share/doc/rsyslog-7.4.7/pgsql-createDB.sql **导入表到数据库Syslog**
* 通过psql -U rsyslog -d Syslog登录数据库,\dt 查看表，owner应该是rsyslog，如不是则需管理员手动修改

## 配置rsyslog 

编辑 /etc/rsyslog.conf文件

	$ModLoad ompgsql 	#载入psql驱动模块
	#*.*	   :ommysql:localhost,Syslog,rsyslog,123456
	*.*        :ompgsql:127.0.0.1,Syslog,rsyslog,123456 

	#mail.*     :ommysql:Syslog,rsyslog,123456
	# Provides UDP syslog reception
	$ModLoad imudp
	$UDPServerRun 514	#开启udp端口监听，意思是通过udp端口接收客户端发过来的日志

	# Provides TCP syslog reception
	$ModLoad imtcp
	$InputTCPServerRun 514  #开启tcp端口监听

**主意#号后是注释，可以不写**

## 检查效果

1. 重启 rsyslog服务
2. psql -U rsyslog -d Syslog 登录postgresql数据库
3. select * from systemevents;

如能查询到数据库中有日志产生，则配置成功

## 配置LOGANALYZER

* LOGANALYZER是用php写的一个web日志分析工具
* 要使用postgresql，需要安装postgre的php扩展

	yum install php-pgsql  php-common

##开始安装LOGANALYZER

下载loganalyzer-'version'.tar.gz并解压缩**
LOGANALYZER的安装实际上是配置，在config.php中配置数据源，用户账户等。
因此config.php需要可写，如不可写，则需要配置selinux和firewall。

	rsync -a loganalyzer-'version'/src/ /var/www/html/{"your web app directry"}
	cd /var/www/html/{"your web app directry"}
	touch config.php
	chmod 666 config.php

**打开安装的浏览器，地址栏输入localhost/loganalyzer/install.php开始安装**
**step1、step2点击next跳过即可**
**LOGANALYZER不支持在postgre上创建登录用户，所以step3中默认配置，点击next直接到step7**
**Source Type中选择数据源，这里选Database(PDO)**
**Database Storage Engine选择PostgreSQL**
**其他配置主机=localhost,数据库名=Syslog，数据库用户=rsyslog，密码123456，数据库表SystemEvents**

配置完成后点下一步、finish。完成配置后自动跳转到index页面，此时已经可以看到日志信息了
