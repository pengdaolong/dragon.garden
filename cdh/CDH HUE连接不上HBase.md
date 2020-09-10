## 问题描述
> CDH中创建Hbase,Hbase 服务正常但是在hue中连接不上Hbase 

### 解决方案
1. 确认 hbase 配置 hbase.thrift.support.proxyuser   hbase.regionserver.thrift.http 是否勾上
2. 确认 core-site.xml 中 HBase 被授权代理

```
<property>
			<name>hadoop.proxyuser.hue.hosts</name>
			<value>*</value>
			</property>
			<property>
			<name>hadoop.proxyuser.hue.groups</name>
			<value>*</value>
			</property>
			<property>
			<name>hadoop.proxyuser.hbase.hosts</name>
			<value>*</value>
			</property>
			<property>
			<name>hadoop.proxyuser.hbase.groups</name>
			<value>*</value>
			</property>
```
![](http://134.175.37.166/upload/2019/1/hue连接不上hbase20190218175530991.png)

3.  检查HUE在hue.ini文件中指定的HBASE的本地配置目录
```
[hbase]
hbase_conf_dir={{HBASE_CONF_DIR}}
thrift_transport=buffered
```
[](http://134.175.37.166/upload/2019/1/hueini20190218175531231.png)
4.  重启服务