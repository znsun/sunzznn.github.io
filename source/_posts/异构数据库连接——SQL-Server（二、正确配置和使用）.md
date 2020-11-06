title: 异构数据库连接——SQL Server（二、正确配置和使用）
author: sunzn
tags:
  - 技能
categories:
  - 数据库
date: 2019-10-22 20:31:00
---
*注：以下文章截图演示环境均为SQL Server2014 Express版*

## 1.准备工作

以在SQL Server环境link MySQL数据库为例，首先要在SQL Server服务器环境安装MySQL的ODBC驱动，这里放上地址：https://dev.mysql.com/downloads/connector/odbc/

官网提供64位和32位的解压版和安装版，高版本是可以兼容低版本的MySQL数据库的，可以按照各自需要进行选择；如果是解压版的话需要解压后手动配置环境变量。

## 2.ODBC数据源配置
1.选择“控制面板—管理工具—数据源(ODBC)”，也可以手动在Windows菜单下“Windows 管理工具”中找到“ODBC数据源”。
![upload successful](/2020/10/11/异构数据库连接——SQL-Server（二、正确配置和使用/pasted-1.png)
2.在“系统DSN”页签下点击添加按钮，选择“MYSQL ODBC Unicode Driver”。
![upload successful](/2020/10/11/异构数据库连接——SQL-Server（二、正确配置和使用/pasted-2.png)
3.创建完成后进行数据源的配置，参数说明如下所示。
* Data Source Name:数据源名称
* Decription:描述,随便写
* Server:MySQL服务器的IP
* Port:MySQL数据库的端口,默认的是3306
* User:账号，MySQL默认是root
* Password:密码
* Database:选择需要链接的数据库实例
![upload successful](/2020/10/11/异构数据库连接——SQL-Server（二、正确配置和使用/pasted-3.png)
4.配置完成后点击“Test”提示“Connection Successful”表示成功，保存即可。

## 3.连接服务器配置
1.打开SQL Server数据库管理器,找到链接服务器.创建链接服务器,如下图所示，只用填写常规选项即可。这里需要注意：访问接口要选择“Microsoft OLE DB Provider for ODBC Drivers”。
![upload successful](/2020/10/11/异构数据库连接——SQL-Server（二、正确配置和使用/pasted-5.png)
2.安全性这里要设置MySQL的用户名和密码。
![upload successful](/2020/10/11/异构数据库连接——SQL-Server（二、正确配置和使用/pasted-6.png)

## 4.一些基本使用和注意点
1.异构有两种查询方式，	正常查询方法为“...”的查询：
>SELECT * FEOM [链接服务器名]…(一个.表示一级目录).[数据库名].[表名]

但是在join多个异构表时考虑到效率问题，应该使用openquery的语法，保证在目标数据库中先将数据查询出来，再通过dblink传输：
>SELECT * FOMR OPENQUERY ( 链接服务器名称,'查询sql' )

2.另外在设置链接服务器时可以采用SQL语法的方式快速生成：
>EXEC  sp_addlinkedserver  
@server='TOSMES',   --链接服务器别名  
@srvproduct='',  
@provider='SQLOLEDB',  
@datasrc='172.16.1.81',  --要访问的的数据库所在的服务器的ip  
@catalog= 'SMES_Test_25111' --访问的数据库名  
GO

>EXEC sp_addlinkedsrvlogin  
'TOSMES',                  --链接服务器别名  
'false',   
 NULL,  
'sa',                     --要访问的数据库的用户  
'Bm2900'                --要访问的数据库，用户的密码  
GO  

3.OPENQUERY因为是调用字符串的方式，所以存在很多使用小技巧和问题，比如字符串内的长度超过8000，或者如果想要传参应该怎么处理，这个我们放到下一章单独来讲。