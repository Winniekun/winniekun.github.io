## 基础介绍

>一个分布式的流平台
>
>基于zookeeper的分布式消息系统
>
>kafka具有高吞吐率、高性能、实时及高可靠等特点



![image-20210315130137621](https://i.loli.net/2021/03/15/6fqFmpez8cTSHPy.png)









![image-20210314230338303](https://i.loli.net/2021/03/14/yubIUGgjwEZOsKW.png)





![image-20210314230559232](https://i.loli.net/2021/03/14/kV5X7MpGESd3TrJ.png)





## 基础架构

![image-20210314231732164](https://i.loli.net/2021/03/14/BRPvh5IEK2qGbAc.png)





## Kafka Producer 客户端

![image-20210315181412381](https://i.loli.net/2021/03/15/p6zcbCmn2JHtiVv.png)

以上能够做到自定义的只有**分区器**了

```java
public class PartitionSample implements Partitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        /**
         * key 格式
         * key-1
         * key-2
         * key-3
         * ...
         */
        String keyStr = key + "";
        String keyIntSub = keyStr.substring(4);
        System.out.println("keyStr: " + keyStr + ", keyIntSub: " + keyIntSub);
        int number = Integer.parseInt(keyIntSub);
        return number % 2;
    }

    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> map) {

    }
}
```

之后在配置里面添加自定义的partition即可

```java
properties = new Properties();
properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "47.95.196.129:9092");
properties.put(ProducerConfig.ACKS_CONFIG, "all");
properties.put(ProducerConfig.RETRIES_CONFIG, "0");
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, "16384");
properties.put(ProducerConfig.LINGER_MS_CONFIG, "1");
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, "33554432");
properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
// partition
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "com.wkk.kafka.producer.PartitionSample");
```



### 消费保障

![image-20210315184718517](https://i.loli.net/2021/03/15/sq83ORSmwhTPIQL.png)



通过如下配置进行确定：

```
ProducerConfig.ACK_CONFIG, "all"/"0"/"1"
```

1. 最多一次

   收到0-1次

   **意思：**

   对于producer而言，仅仅确保自己把消息发送出去，不会去确定消息的确是被接收了。

2. 至少一次

   **意思：**

   对于producer而言，在确保自己把消息发送出去的基础上，还要确定消息是被接收了，所以会不停的发，直到producer收到消息已经被收到确认。

   收到1到多次

3. 正好一次

   有且仅有1次

   最严格的校验

   

## 常见面试题目

吞吐量高，但是不保证消息有序性

### 和其他消息中间件的区别

1. 概念

   分布式流平台

2. kafka特性一

   提供发布和订阅的功能

3. kafka特性二

   吞吐量高但不保证消息有序性

串讲如下：

> 定义不同，kafka本身是分布式的流处理平台，只不过刚好其提供了发布订阅以及Topic的支持，所以它可以提供和消息队列相同的特性，但是kafka自身是提供了非常高的吞吐量，所以从整体来看相比于其他的消息中间件，能够吞吐量要大得多。可是问题在于，kafka没有提供消息有序性的保障（不同的partition）。同时kafka可以消费历史的数据，这也是和其他的消息队列不同的地方

### 应用场景

1. 日志收集或流式系统

2. 消息系统

   **无法保证消息有序**

3. 用户活动跟踪或运营指标监控



### Kafka吞吐量为什么大

**类似：** 速度为什么快

>1. 读取消息内容，并存储磁盘
>2. 当consumer消费的时候，将消息从磁盘中消费出来
>
>以上的两个步骤牵扯到了
>
>1. 日志存储
>2. 消息推送
>3. 消息消费
>4. 消息检索

#### 日志顺序读写和快速检索

#### partition机制

可以并行

#### 批量的发送接收及数据压缩 

#### 通过sendfile实现**零拷贝**原则





## kafka底层原理之日志

1. 日志以partition为单位进行存储
2. 日志目录格式为topic+数字
3. 日志文件格式是一个“日志条目”序列
4. 每条日志消息由4字节整形与N字节消息组成



### 日志分段

1. 每个partition的日志会分为N个大小相等的segment中
2. 每个segment中消息数量不一定相等
3. 每个partition只支持顺序读写
   1. 磁盘中的顺序读写速度快



### segment存储结构

1. partition会将消息添加到最后一个segment上
2. 当segment达到一定阈值会flush到磁盘上
3. segment文件会分为index + data



### 日志读操作

1. 首先需要在存储的数据中找出segment文件
2. 然后通过全局offset计算出segment中的offset
3. 通过index中的offset寻找具体数据内容

### 日志写操作

1. 日志允许串行的追加消息到文件最后
2. 当日志文件达到阈值则滚动到新文件上

### 零拷贝

参见Java核心技术面试36讲之12



## kafka消费者和消费者组

1. 消费者组是kafka消费的单位
2. 单个partition只能由消费者组中某个消费者消费
3. 消费者组中的消费者可以消费多个partition



## kafka如何保证顺序性

1. kafka的特性只支持单partitino有序

   1. 影响吞吐量和消费者数量

2. 使用key + offset 可以做到业务有序性

   业务较为独特

   1. 订单
   2. 换电流程等
      1. 订单号唯一，视为key
      2. 状态不同，判断顺序

## topic删除背后的故事

![image-20210315195858986](https://i.loli.net/2021/03/15/1AoajDxY78n6OBw.png)

