1. 安装jdk1.8以上
2. 创建数据库和角色
```
create database jira charset utf8 COLLATE utf8_bin;
create user jira@'%' identified by '123456';
grant all privileges on *.* to jira@'%';
```
3. 下载bin
```
wget https://downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-7.3.8-x64.bin(7.12.3也能成功亲测)
```
4. 下载破解jar [点击下载 提取码brye ](https://pan.baidu.com/s/12_aRvWs9ldLJKS8MI8rd7w) 
5. 执行下载的bin
```
chmod  +x  atlassian-jira-software-7.3.8-x64.bin
./atlassian-jira-software-7.3.8-x64.bin
![file](/upload/2019/3/image-15556611622762019041916060230.png)
```
6. 先关闭jira 
```
 /etc/init.d/jira stop
```
7. 替换文件
```
cd  /opt/atlassian/jira/atlassian-jira/WEB-INF/lib/
其中atlassian-extras-3.2.jar是用来替换原来的atlassian-extras-3.2.jar文件，用作破解jira系统的。
而mysql-connector-java-5.1.39-bin.jar是用来连接mysql数据库的驱动软件包。
 /etc/init.d/jira start
```
8. 配置服务
```
 ![file](/upload/2019/3/image-155566136504920190419160925677.png)
选择其他数据库（生产中）
![file](/upload/2019/3/image-155566140706220190419161007468.png) 
数据库的连接配置是再 /var/atlassian/application-data/jira/dbconfig.xml
![file](/upload/2019/3/image-155566143419820190419161034117.png)
![file](/upload/2019/3/image-155566144473920190419161044595.png)  
用你的邮箱进行登录
![file](/upload/2019/3/image-155566154173420190419161221250.png)
成功生成时会自动跳转到原来的页面中 点击下一步
对管理员账号进行设置
![file](/upload/2019/3/image-155566157800420190419161258788.png)
![file](/upload/2019/3/image-155566159270820190419161312904.png)
```
9.