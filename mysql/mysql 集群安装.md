管理节点（负责管理）
管理节点安装
安装管理节点（192.168.153.3）
[root@localhost /]# groupadd mysql
[root@localhost /]#  useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64 mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db –user=mysql
管理节点配置
[root@localhost ~]#  mkdir /var/lib/mysql-cluster
[root@localhost ~]# cd /var/lib/mysql-cluster
[root@localhost mysql-cluster]# vi  /var/lib/mysql-cluster/config.ini
在config.ini 中添加以下内容:
[NDBD DEFAULT]
NoOfReplicas=1
[TCP DEFAULT]
portnumber=3306 
[NDB_MGMD]
#设置管理节点服务器
HostName=192.168.153.3
DataDir=/var/mysql/data
[NDBD]
#设置存储节点服务器(NDB节点)
HostName=192.168.153.4
DataDir=/var/mysql/data
[NDBD]
#第二个NDB节点
HostName=192.168.153.5
DataDir=/var/mysql/data
[MYSQLD]
#设置SQL节点服务器 
HostName=192.168.153.6
管理节点启动
[root@localhost ~]# /usr/local/mysql/bin/ndb_mgmd -f /var/lib/mysql-cluster/config.ini
[root@localhost ~]# mkdir /var/mysql/logs
看到tcp 0 0 0.0.0.0:1186开放说明启动正常
开启管理节点服务器的1186端口
管理节点检验
执行以下操作：
[root@localhost /]# ndb_mgm     // 管理节点 
![file](/upload/2020/7/image-159711656872420200811112928594.png)
管理节点关闭
[root@localhost /]# /usr/local/mysql/bin/ndb_mgm -e shutdown
数据节点
数据节点安装
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64 mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db --user=mysql
[root@localhost mysql]# cp support-files/my-medium.cnf /etc/my.cnf
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
数据节点配置（存储数据）
对数据节点进行配置，执行以下操作：
[root@localhost mysql]# mkdir /var/mysql/data
[root@localhost mysql]# mkdir /var/mysql/logs
[root@localhost mysql]# vi /etc/my.cnf
向文件追加以下内容：
[MYSQLD]
ndbcluster
ndb-connectstring=192.168.153.3
[MYSQL_CLUSTER]
ndb-connectstring=192.168.153.3
[NDB_MGM]
connect-string=192.168.153.3
数据节点启动
启动此处时，管理节点服务器防火墙必须开启1186,3306端口。
注意：只是在第一次启动或在备份／恢复或配置变化后重启ndbd时，才加–initial参数！
第一次启动如下：
[root@localhost mysql]# /usr/local/mysql/bin/ndbd --initial
2013-01-30 13:43:53 [ndbd] INFO     -- Angel connected to '192.168.153.3
:1186'
2013-01-30 13:43:53 [ndbd] INFO     -- Angel allocated nodeid: 2
正常启动方式：
[root@localhost mysql]# /usr/local/mysql/bin/ndbd
数据节点关闭
[root@localhost /]# /etc/rc.d/init.d/mysqld stop
或者
[root@localhost mysql]# /etc/init.d/mysql stop
Shutting down MySQL.. SUCCESS!

/usr/local/mysql/bin/mysqladmin -uroot shutdown
SQL节点安装（sql执行）
SQL节点安装
SQL节点和存储节点(NDB节点)安装相同，都执行以下操作；
SQL节点配置
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd mysql -g mysql
[root@localhost /]# cd /usr/local
[root@localhost local]# tar -zxv -f mysql-cluster-gpl-7.2.6-linux2.6-x86_64.tar.gz
[root@localhost local]# mv mysql-cluster-gpl-7.2.6-linux2.6-x86_64 mysql
[root@localhost local]# chown -R mysql:mysql mysql
[root@localhost local]# cd mysql
[root@localhost mysql]# scripts/mysql_install_db --user=mysql
[root@localhost mysql]# cp support-files/my-medium.cnf /etc/my.cnf
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
执行以下操作：
[root@localhost mysql]# mkdir /var/mysql/data     //创建存储数据的文件夹
[root@localhost mysql]# mkdir /var/mysql/logs     //创建存储日志的文件夹
[root@localhost mysql]# vi /usr/local/mysql/my.cnf  //修改配置文件
追加以下内容：
[MYSQLD]
ndbcluster
ndb-connectstring=192.168.153.3
[MYSQL_CLUSTER]
ndb-connectstring=192.168.153.3
[NDB_MGM]
connect-string=192.168.153.3
SQL节点启动
执行以下操作：
[root@localhost mysql]# service mysqld start
Starting MySQL.. SUCCESS!
SQL节点关闭
最直接的方式：
[root@localhost mysql]# /usr/local/mysql/bin/mysqladmin -uroot shutdown

[root@localhost /]# /etc/rc.d/init.d/mysqld stop
或者
[root@localhost mysql]# /etc/init.d/mysql stop
Shutting down MySQL.. SUCCESS!
![file](/upload/2020/7/image-159711654027720200811112900217.png)
![file](/upload/2020/7/image-159711655126320200811112912332.png)
Mysql-sql节点报错，因为环境mysql没卸载干净，第二次安装时一定要把文件删干净！！！
见表一定要加ENGINE=NDB
