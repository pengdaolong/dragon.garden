1、消费的并行度   director  task 的个数取决于partition
> 1个partition的吞吐 大概10m/s  
> partition 不是越多越好 多了  会占用内存  打开过多的文件句柄{客户端producer有个参数batch.size默认为 16KB。它会为每个分区缓存消息，一旦批次数满了后，将消息批量发出。一般来说，这个设计是用于提升吞吐性能的。但是由于这个参数是partition级别的，如果分区数越多，这部分缓存所需的内存占用也会越多。

假如有 10000 个分区，按照默认配置，这部分缓存就要占用约 157MB 的内存。而consumer端呢？抛开拉取数据所需的内存不说，单说线程的开销。如果还是 10000 个分区，同时consumer线程数要匹配分区数的话(大部分情况下是最佳的消费吞吐量配置)，那么在consumer client就要创建 10000 个线程，也需要创建大约 10000 个Socket去获取分区数据，这里面的线程切换的开销本身就已经不容小觑了。

服务器端的开销也不小，如果阅读kafka源码的话就会发现，服务器端的很多组件在内存中维护了partition级别的缓存，比如controller，FetcherManager等，因此分区数越多，这种缓存的成本就越大。}
>分区数 大概=broker 数量 *  3/6 

2、序列化
> java的序列化 要很久
> 建议用kryo 

3、限流和反压
>每秒如果从 kafka 拉去过多的数据量 如 10000000000  消耗的资源就很多了  再计算 太耗资源 拉取适量的数据
>限流 spark.streaming.kafka.maxRatePerPartition   每个分区取多少条数据
>反压 动态调整数量 无需人工干预   spark.streaming.backpressure.enabled  默认false
RateController
RateEstimator
RateLimiter   
pidRateController
4、cpu 空转
> 流  >task 没有接收到数据 
> spark.locality.wait 3s
5、不要再代码中判断表存不存在
>直接把表建好 
6、推测执行
>task 失败  把task调到其他的机器上执行     
spark.speculation :默认false。是否开启推测执行。
spark.speculation.interval :默认100ms。多久检查一次要推测执行的Task。
spark.speculation.multiplier :默认1.5。一个Stage中，运行时间比成功完成的Task的运行时间的中位数还慢1.5倍的Task才可能会被推测执行。
spark.speculation.quantile: 默认0.75。推测的分位数。即一个Stage中，至少要完成75%的Task才开始推测。(0.9比较好)