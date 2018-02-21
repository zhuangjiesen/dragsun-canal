# Canal 部署与运行


### github 地址
[canal github ](https://github.com/alibaba/canal)

教程和运行都很全面

### 本机上的 mysql 版本： 5.7.21-log

### 原理：
```
就是启动一个(模拟) MySQL 的slave 服务，获取mysql 的row 类型的 binlog ，然后就可以对数据库的变化进行一些操作

基于日志增量订阅&消费支持的业务：


数据库镜像
数据库实时备份
多级索引 (卖家和买家各自分库索引)
search build
业务cache刷新
价格变化等重要业务消息
keyword：数据库同步，增量订阅&消费。

```


### 我研究的初衷主要是为了cache 的刷新

## 部署canal：

### 部署canal-server：
####（1）开启MySQL的binlog功能，并配置binlog模式为row。
在my.cnf 加入如下：

```
[mysqld]  
log-bin=mysql-bin #添加这一行就ok  
binlog-format=ROW #选择row模式  
server_id=1 #配置mysql replaction需要定义，不能和canal的slaveId重复 
```
 
####（2）在mysql中 配置canal数据库管理用户，配置相应权限（repication权限）
```
CREATE USER canal IDENTIFIED BY 'canal';    
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';  
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;  
FLUSH PRIVILEGES;  
（3）下载canal https://github.com/alibaba/canal/releases 
 解压到相应文件夹
[html] view plain copy
tar -zxvf canal canal  
canal 文件目录结构
[html] view plain copy
drwxr-xr-x 2 jianghang jianghang  136 2013-02-05 21:51 bin  
drwxr-xr-x 4 jianghang jianghang  160 2013-02-05 21:51 conf  
drwxr-xr-x 2 jianghang jianghang 1.3K 2013-02-05 21:51 lib  
drwxr-xr-x 2 jianghang jianghang   48 2013-02-05 21:29 logs  
```

#### 修改配置 instance.properties

```
vim canal/conf/example/instance.properties  
```


```
#################################################  
## mysql serverId  
canal.instance.mysql.slaveId = 1234  
  
# position info，需要改成自己的数据库信息  
canal.instance.master.address = 127.0.0.1:3306   
canal.instance.master.journal.name =   
canal.instance.master.position =   
canal.instance.master.timestamp =   
  
#canal.instance.standby.address =   
#canal.instance.standby.journal.name =  
#canal.instance.standby.position =   
#canal.instance.standby.timestamp =   
  
# username/password，需要改成自己的数据库信息  
canal.instance.dbUsername = canal    
canal.instance.dbPassword = canal  
canal.instance.defaultDatabaseName = canal_test  
canal.instance.connectionCharset = UTF-8  
  
# table regex  
canal.instance.filter.regex = .*\\..*  
  
#################################################  
```

#### 然后cd到bin目录  启动和停止canal-server
#### 启动:

```
./startup.sh  
```
#### 停止

```
./stop.sh  
```
#### 验证启动状态，查看log文件
```
vim canal/log/canal/canal.log  
```
```
2014-07-18 10:21:08.525 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.  
2014-07-18 10:21:08.609 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.12.109.201:11111]  
2014-07-18 10:21:09.037 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......  
```

上述日志信息显示启动canal成功


### 或者下载源码

#### 版本：v1.0.24
```
https://github.com/alibaba/canal/releases
```

#### 运行或者开发：
```
入口：
模块： deployer 
类： CanalLauncher.java 
配置文件也是在这模块下一样配置

```



### 客户端程序

```
在 JavaProCustomer 项目下，有基于spring 的 canal 小框架

```


