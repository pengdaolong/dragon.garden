### spark thriftServer 目前没有实现ha，可能会出现单点故障的风险，现在通过修改源码将这个服务变成高可用的实现主要的步骤：
> 1、下载源码
> 2、在hive-thriftserver2开启服务时将信息注册的zk中
> 3、在服务结束时将注册信息从zk中移除

### 添加依赖
1. 在 spark\pom.xml modules中 添加
```
    <module>sql/hive-thriftserver</module>
```

### 修改源码

1. 先打开 hiveThriftServer2.scala
```
  /**
    * 初始化方法
    * @param hiveConf
    */
  // 将hiveConf设置成全局变量 后面要用到配置
  var hiveConf : HiveConf = _
  override def init(hiveConf: HiveConf) {
    this.hiveConf = hiveConf
    val sparkSqlCliService = new SparkSQLCLIService(this, sqlContext)
    setSuperField(this, "cliService", sparkSqlCliService)
    addService(sparkSqlCliService)
```
2. 在thriftserver2start时 将注册信息注册到zk中（spark\sql\hive-thriftserver\src\main\java\org\apache\hive\service\server\HiveServer2.java）
```
  /**
    *  开始的方法 需要像zk注册信息
    */
  override def start(): Unit = {
    super.start()
    started.set(true)
    // add metadata to zk
    if(this.hiveConf.getBoolVar(HiveConf.ConfVars.HIVE_SERVER2_SUPPORT_DYNAMIC_SERVICE_DISCOVERY)){
	//反射调用 HiveServer2 addServerInstanceToZooKeeper 方法自己实现
      invoke(classOf[HiveServer2], this, "addServerInstanceToZooKeeper",
        classOf[HiveConf]->this.hiveConf)
    }
  }
```
```
/**
   * 注册thriftserver信息到zookeeper
   * @param hiveConf
   */
  private CuratorFramework zookeeperClient;
  private String znodePath;
  private void addServerInstanceToZooKeeper(HiveConf hiveConf) throws Exception {
    String zookeeperEnsemble = ZooKeeperHiveHelper.getQuorumServers(hiveConf);
    //连接zookeeper 超时时间
    int zkSessionTimeOut = (int)hiveConf.getTimeVar(HiveConf.ConfVars.HIVE_ZOOKEEPER_SESSION_TIMEOUT, TimeUnit.MILLISECONDS);
    //hive连接zookeeper的等待时间
    int baseSleepTime =
            (int) hiveConf.getTimeVar(HiveConf.ConfVars.HIVE_ZOOKEEPER_CONNECTION_BASESLEEPTIME,
                    TimeUnit.MILLISECONDS);
    //链接zk的最大重试次数
    int maxRetries  =(int)hiveConf.getIntVar(HiveConf.ConfVars.HIVE_ZOOKEEPER_CONNECTION_MAX_RETRIES);
    zookeeperClient= CuratorFrameworkFactory.builder().connectString(zookeeperEnsemble).sessionTimeoutMs(zkSessionTimeOut)
            .retryPolicy(new ExponentialBackoffRetry(baseSleepTime,maxRetries)).build();

    zookeeperClient.start();
    //从hive-site.xml中获取hive.server2.zookeeper.namespace的配置信息
    String rootNamespace = hiveConf.getVar(HiveConf.ConfVars.HIVE_SERVER2_ZOOKEEPER_NAMESPACE);
    try {
      //创建一个路径用来存hiveThriftServer2的metadata
      zookeeperClient.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
              .forPath(ZooKeeperHiveHelper.ZOOKEEPER_PATH_SEPARATOR+rootNamespace);
      LOG.info("created the root namespark:"+rootNamespace+"no zookeeper for hiveThriftServer2");
    } catch (Exception e) {
      e.printStackTrace();
      LOG.info("Unable to create the root namespark:"+rootNamespace+"no zookeeper for hiveThriftServer2");
    }
    //得到thriftserver2的ip:port
    String serverIPAddressAndPort = getServerIPAddressAndPort();
    try{
      //拼接hiveThriftServer2 需要保存的数据信息
      String pathPrefix=ZooKeeperHiveHelper.ZOOKEEPER_PATH_SEPARATOR+rootNamespace+
              ZooKeeperHiveHelper.ZOOKEEPER_PATH_SEPARATOR+"server:"+serverIPAddressAndPort+";"
              +"version:"+ HiveVersionInfo.getVersion()+";"+"date:"+HiveVersionInfo.getDate()+";"
              +"sequence=";
      //得到znode
      byte[] znodeDataUTF8 = serverIPAddressAndPort.getBytes(Charset.forName("UTF-8"));
      znode=new PersistentEphemeralNode(zookeeperClient,
              PersistentEphemeralNode.Mode.EPHEMERAL_SEQUENTIAL,pathPrefix,znodeDataUTF8);
      znode.start();
      // We'll wait for 120s for node creation
      long znodeCreationTimeout = 120;
      if (!znode.waitForInitialCreate(znodeCreationTimeout, TimeUnit.SECONDS)) {
        throw new Exception("Max znode creation wait time: " + znodeCreationTimeout + "s exhausted");
      }
      setDeregisteredWithZooKeeper(false);
      znodePath = znode.getActualPath();
      // TODO 添加zk的watch，如果服务不见了，需要第一时间watche到
      if (zookeeperClient.checkExists().usingWatcher(new DeRegisterWatcher()).forPath(znodePath) == null) {
        // No node exists, throw exception
        throw new Exception("Unable to create znode for this HiveServer2 instance on ZooKeeper.");
      }
      LOG.info("Created a znode on ZooKeeper for HiveServer2 uri: " + serverIPAddressAndPort);
    }catch (Exception e){
      if(znode!=null){
        znode.close();
      }
      throw e;
    }
  }
   /**
   * 得到thriftServer2的ip:port
   * @return
   * @throws Exception
   */
  private String getServerIPAddressAndPort() throws Exception {
    if((null==thriftCLIService)||null==thriftCLIService.getServerIPAddress()){
      throw new Exception("unable get regist server add ;");
    }
    return getHiveHost()+":"+thriftCLIService.getPortNumber();
  }
  private String getHiveHost() {
    HiveConf hiveConf = thriftCLIService.getHiveConf();
    String hiveHost = hiveConf.getVar(HiveConf.ConfVars.HIVE_SERVER2_THRIFT_BIND_HOST);
    if (hiveHost != null && !hiveHost.isEmpty()) {
      return hiveHost;
    } else {
      return thriftCLIService.getServerIPAddress().getHostName();
    }
  }
    /**
   * 控制是否需要重新在zookeeper上注册HiveServer2
   * */
  private boolean deregisteredWithZooKeeper = false;
  private void setDeregisteredWithZooKeeper(boolean deregisteredWithZooKeeper) {
    this.deregisteredWithZooKeeper = deregisteredWithZooKeeper;
  }
```
3. thriftserver2退出服务时(hiveThriftServer2.scala)
```
  override def stop(): Unit = {
    if (started.getAndSet(false)) {
	//add  
      if (this.hiveConf.getBoolVar(
        ConfVars.HIVE_SERVER2_SUPPORT_DYNAMIC_SERVICE_DISCOVERY)) {
        invoke(classOf[HiveServer2], this, "removeServerInstanceFromZooKeeper")
      }
       super.stop()
    }
  }
```
```
修改hiveServer2.java
  //移除znode，代表当前程序关闭
  private void removeServerInstanceFromZooKeeper() throws Exception {
    setDeregisteredWithZooKeeper(true);
    if (znode != null) {
      znode.close();
    }
    zookeeperClient.close();
    LOG.info("Server instance removed from ZooKeeper.");
  }
```

### 打包源码（在git bash 中运行如下代码）
```
时间很久 要外网 
build/mvn -Pyarn -Phadoop-2.6 -Dhadoop.version=2.6.0 -Phive -Phive-thriftserver -Dscala-2.11 -DskipTests clean package
![file](/upload/2020/7/image-159711219524120200811101635125.png)
```
### 替换jar
```
spark\sql\hive-thriftserver\target\spark-hive-thriftserver_2.11-2.4.8.jar 替换集群中的
```

### 修改配置文件 hive-site
```
<property>
    <name>hive.server2.support.dynamic.service.discovery</name>
    <value>true</value>
</property>
<property>
    <name>hive.server2.zookeeper.namespace</name>
    <value>zk_hiveserver2</value>
</property>

<property
    <name>hive.zookeeper.quorum</name>
    <value>cdh1:2181,cdh2:2181,cdh3:2181</value>
</property>
<property>
    <name>hive.zookeeper.client.port</name>
    <value>2181</value>
</property>
<property>
    <name>hive.server2.thrift.bind.host</name>
    <value>cdh1</value>
</property>
```
### 修改启动脚本
```
/opt/cloudera/parcels/CDH/lib/spark/sbin# vim spark-daemon.sh 
# 屏蔽以下脚本###########################################
# if [ -f "$pid" ]; then
#   TARGET_ID="$(cat "$pid")"
#   if [[ $(ps -p "$TARGET_ID" -o comm=) =~ "java" ]]; then
#     echo "$command running as process $TARGET_ID.  Stop it first."
#     exit 1
#   fi
# fi
########################################################
```
###  启动服务
```
sbin/start-thriftserver.sh \
--master yarn \
--conf spark.driver.memory=1G \
--executor-memory 1G \
--num-executors 1 \
--hiveconf hive.server2.thrift.port=10001
```
### 链接服务
```
!connect jdbc:hive2://cdh1:2181,cdh2:2181,cdh3:2181/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=zk_hiveserver2
```