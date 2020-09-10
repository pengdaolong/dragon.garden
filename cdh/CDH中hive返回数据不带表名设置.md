hive-site.xml 的 HiveServer2 高级配置代码段（安全阀）
添加
```
<property>
<name>hive.resultset.use.unique.column.names</name>
<value>false</value>
</property>
```