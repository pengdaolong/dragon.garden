```
<property>
<name>oozie.processing.timezone</name>
<value>GMT+0800</value>
</property>
<property>
<name>oozie.service.HadoopAccessorService.nameNode.whitelist</name>
<value> </value>
</property>
<property>
<name>oozie.service.HadoopAccessorService.jobTracker.whitelist</name>
<value> </value>
</property>
<property>
<name>oozie.action.shell.setup.hadoop.conf.dir.log4j.content</name>
<value>
log4j.rootLogger=INFO,console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
</value>
</property>
<property>
<name>oozie.use.system.libpath</name>
<value>true</value>
</property>
<property>
<name>oozie.libpath</name>
<value>${nameNode}/user/oozie/share/lib</value>
</property>
```