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

# PostgreSQL

## 安装

	yum install postgresql postgresql-server

正常情况下，安装完成后，PostgreSQL服务器会自动在本机的5432端口开启。
初次安装后，默认生成一个名为postgres的数据库和一个名为postgres的数据库用户。**这里需要注意的是，同时还生成了一个名为postgres的Linux系统用户。**

## 创建用户

通过命令su - postgres进入postgresql管理账户

	psql 			/*登录PostgreSQL控制台,相当于命令psql -U postgres -d postgres*/
	\password postgres 	/*设置密码*/
	create role rsyslog
## 创建数据库

	create database "Syslog" owner=rsyslog (**数据库名是大写字母必须要用双引号括起来**)
	grant all on database Syslog to rsyslog /**为用户rsyslog授权**/
## 配置rsyslog
* yum install rsyslog-pgsql 安装rsyslog的postgresql驱动模块


