1.  安装JDK 1.7以上。
2.  在mysql中创建对应得数据库和用户。
```
create database confluence charset utf8  COLLATE utf8_bin ;
create user 'confluence'@'%' identified by 'YOUR PASSWORD';
grant all privileges on *.* to 'confluence'@'%';
```
3. 查看confluence的所有版本 [confluence连接](https://www.atlassian.com/software/confluence/download-archives) 
4. 下载 可执行文件
```
 wget https://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-6.9.1-x64.bin
```
5. 修改文件权限
```
chmod +x atlassian-confluence-6.9.1-x64.bin  
```
6. 下载破解文件 [破解文件地址 提取码nnxr](http://https://pan.baidu.com/s/1AuW4YnDollCH5qIEs0b2ZA)
7. 执行./atlassian-confluence-6.9.1-x64.bin
![file](/upload/2019/3/image-155565924178720190419153401318.png)
8.通过hostip:PORT就可以访问到界面了
9.记录下server-id
![file](/upload/2019/3/image-15556593446422019041915354458.png)
10. 停止运行
```
/etc/init.d/confluence stop  
```
11.替换jar包
```
cd /opt/atlassian/confluence/confluence/WEB-INF/lib
rm -fr atlassian-extra* 
将前面下载的文件atlassian-extras-3.2.jar（改名为atlassian-extras-decoder-v2-3.3.0.jar） mysql-connector-java-5.1.39-bin.jar，拷贝进去。
```
12.再次启动
```
/etc/init.d/confluence start 
```
13.生成密钥
```
cmd  进入到 刚刚下载的破解文件里
![file](/upload/2019/3/image-155565959674520190419153956269.png)
然后在黑窗口执行  java -jar confluence_keygen.jar
![file](/upload/2019/3/image-155565963045720190419154030230.png)
点击gen 生成key
```
14.再次访问ip:port页面 将上一步的key拷贝进 License Key中 点击next
15.选择外部数据库
```
这里我使用的mysql 
然后direct jdbc
配置自己的数据库连接 点击next（要一段时间，要生成一些表）。
```
16.初始化一个空的站点  Empty site (两次都选择这个)
17.配置confluence的管理员账号和密码（Manage users and groups within confluence） `可以选择和jira连接，这样就共享账户和密码`  


#### 数据库乱码问题
```
先关闭服务 /etc/init.d/confluence stop
vi  /var/atlassian/application-data/confluence/confluence.cfg.xml
在配置项hibernate.connection.url中指定数据库的最后url的最后添加?useUnicode=true&characterEncoding=utf8&useSSL=true
```
#### 配置MySQL
```
Centos  /etx/my.cnf    ubuntu vim /etc/mysql/mysql.conf.d/mysqld.cnf
在[mysqld]下添加下面配置
innodb_log_file_size=256M
max_allowed_packet=34M
```