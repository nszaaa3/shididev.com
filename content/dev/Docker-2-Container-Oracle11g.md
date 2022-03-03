---
title: Docker-2、容器安装之Oracle 11g r2

date: 2022-03-03 09:25:12

tags:

- Docker
- Oracle
- Database

categories:

- Docker

---

## 吐槽

我还在纠结要不要发布这篇文章, 因为谁能想到2022年了居然还要写Oracle
11g的docker安装方法。我最早是在2017年6月份在笔记里记录的11g的docker，一晃5年过去了，这篇文章还用得上。我在几个群里边问了问，很多企业还在用11g和12c，就像他们还在用MySQL5.6，还在用JDK7，还在坚守CentOS，还在用SVN。

再延伸一下，如果文章会打酱油的话，现在[CSDN](/hexo/img/bazinga.gif)上到处都是长得一模一样的孩子开杂货铺卖酱油了，中文互联网内容在凋零，新互联网人在抱残守缺，不愿输出新内容，进而导致全网性的内容过时、技术过时。

## Oracle 11g

### 容器安装

```shell
docker run  -p 1521:1521 \
-v /opt/docker/oracle/data/oracle:/data/oracle \
--name oracle11 \
-d registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

- -p [xxxx]:1521 将容器中的1521端口映射到宿主机的xxxx端口
    - p - port 端口
- -v [/xxx/xxx]: /data/oracle 将容器中的/data/oracle 目录映射到宿主机的/xxx/xxx目录，在这里我映射到宿主机的/opt/docker/oracle/data/oracle
    - v - volume 卷
- -d 后台运行容器，并返回容器ID
    - d - detach 分离

### 配置环境变量

进入容器内部

```shell
docker exec -it --user root oracle11 /bin/bash
```

安装vim编辑器

```shell
yum install vim
```

编辑 /etc/profile

```shell
vim /etc/profile
```

在尾部添加如下字段

```shell
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2
export ORACLE_SID=helowin
export PATH=$ORACLE_HOME/bin:$PATH
```

### 创建新用户并授权远程登录

```shell
su -oracle
sqlplus /nolog
conn /as sysdba
#修改system用户账号密码；
alter user system identified by system;
#修改sys用户账号密码；
alter user sysidentified by system;
#创建内部管理员账号；
create user YONGHUMING identified by MIMA; 
#授权
grant connect,resource,dba to YONGHUMING;  
 #修改密码规则策略为密码永不过期；
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
#修改数据库最大连接数据；
alter system set processes=1000 scope=spfile; 
exit

```

### 重启数据库使之生效

```shell
sqlplus /nolog
conn /as sysdba
shutdown immediate;
startup;
exit
```

### 修改字符集为 utf-8

```shell
#Server端字符集修改
sqlplus "/as sysdba"
conn /as sysdba;
shutdown immediate;
startup mount;
#将数据库启动到RESTRICTED模式下做字符集更改：
ALTER SYSTEM ENABLE RESTRICTED SESSION;
ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
ALTER SYSTEM SET AQ_TM_PROCESSES=0;
alter database open;
alter database character set INTERNAL_USE UTF8;
shutdown immediate;
startup;
exit;
```

至此，Oracle 11g 的容器安装完成，但是真不希望有人再继续安装11g，转而去安装更高版本，我也会在后续日志中补全12g以及后续LTS版本。
