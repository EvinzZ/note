# java整合kafka

## 1.生产者

### 1.1.引入依赖

```xml
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>2.4.1</version>
        </dependency>
```

### 1.2生产者代码

异步发送

```java
package com.example.demokafka.kafkademo;

import com.alibaba.fastjson.JSON;
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 消息的发送方
 */
public class MyProducer {
    private final static String TOPIC_NAME = "my-replicated-topic";

    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.200.10:9092"); // 集群连接用,分隔
        props.put(ProducerConfig.ACKS_CONFIG, "1");
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        int msgNum = 5;
        final CountDownLatch countDownLatch = new CountDownLatch(msgNum);
        for (int i = 1; i < 10; i++) {
            Order order = new Order((long)i, i);
            ProducerRecord<String, String> producerRecord = new ProducerRecord<>(TOPIC_NAME, order.getOrderId().toString(), JSON.toJSONString(order));
            producer.send(producerRecord, new Callback() {
                @Override
                public void onCompletion(RecordMetadata recordMetadata, Exception e) {
                    if (e != null) {
                        System.out.println("发送消息失败：" + e.getStackTrace());
                    }
                    if (recordMetadata != null) {
                        System.out.println("异步方式发送消息结果：" + "topic_" +recordMetadata.topic() + "| partition-" + recordMetadata.partition() + "|offset-" + recordMetadata.offset());
                    }
                    countDownLatch.countDown();
                }
            });
        }
        countDownLatch.await(5, TimeUnit.SECONDS);
        producer.close();
    }
}




package com.example.demokafka.kafkademo;

public class Order {
    private Long orderId;
    private int count;

    public Order(Long orderId, int count) {
        this.orderId = orderId;
        this.count = count;
    }

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}

```

同步发送

```java
package com.example.demokafka.kafkademo;

import com.alibaba.fastjson.JSON;
import com.sun.org.apache.xpath.internal.operations.Or;
import lombok.SneakyThrows;
import org.apache.kafka.clients.producer.*;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

/**
 * 消息的发送方
 */
public class MyProducer {
    private final static String TOPIC_NAME = "my-replicated-topic";

    @SneakyThrows
    public static void main(String[] args) throws InterruptedException {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.200.10:9092"); // 集群连接用,分隔
        props.put(ProducerConfig.ACKS_CONFIG, "1");
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
        props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        Order order = new Order(1L, 1);
        ProducerRecord<String, String> producerRecord = new ProducerRecord<String, String>(TOPIC_NAME, order.getOrderId().toString(), JSON.toJSONString(order));
        RecordMetadata metadata = producer.send(producerRecord).get();
        System.out.printf("同步方式发送消息结果：topic_%s|partition-%s|offset-%s", metadata.topic(), metadata.partition(), metadata.offset());
        producer.close();
    }
}


package com.example.demokafka.kafkademo;

public class Order {
    private Long orderId;
    private int count;

    public Order(Long orderId, int count) {
        this.orderId = orderId;
        this.count = count;
    }

    public Long getOrderId() {
        return orderId;
    }

    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}

```

### 1.3.生产者中的ack的配置

在同步发送的前提下，生产者在获得集群返回的ack之前会一直阻塞。那么集群什么时候返回ack？此时ack有3个配置：

- `ack = 0`：kafka-cluster不需要任何的broker收到消息，就立即返回ack给生产者。（最容易丢消息，效率最高）
- `ack = 1`（默认）：多副本之间的leader已经收到消息，并把消息写入到本地的log中，才会返回ack给生产者。（性能和安全性最均衡）
- `ack = -1/all`：里面有默认的配置`min.insync.replicas=2`（默认为1，推荐配置大于等于2），此时就需要leader和一个follower同步完后，才会返回ack给生产者（此时集群中有2个已完成数据的接收）。（这种方式最安全，但性能最差）

```java
        props.put(ProducerConfig.ACKS_CONFIG, "1"); // ack配置
        props.put(ProducerConfig.RETRIES_CONFIG, 3); // 重试配置
        props.put(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, 300); // 重试间隔设置
```

### 1.4.消息发送的缓冲区

- kafka默认会创建一个消息缓冲区，用来存放要发送的信息，缓冲区是32M

```java
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
```

- kafka本地线程会去缓冲区中一次拉16k的数据，发送到broker

```java
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
```

- 如果线程拉不到16k的数据，间隔10ms也会将已拉到的数据发到broker

```java
        props.put(ProducerConfig.LINGER_MS_CONFIG, 10);
```

## 2.消费者

```java
package com.example.demokafka.kafkademo;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Arrays;
import java.util.Properties;

/**
 * 消费者
 */
public class MyConsumer {
    private final static String TOPIC_NAME = "my-replicated-topic";
    private final static String CONSUMER_GROUP_NAME = "testGroup";
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "192.168.200.10:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, CONSUMER_GROUP_NAME); // 消费组名
//        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true"); // 是否自动提交offset，默认true
//        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000"); // 自动提交offset的间隔时间
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        /**
         * 当小费主题的是一个新的消费组，或者指定offset的消费方式，offset不存在。那么应该如何消费
         * latest（默认）：只消费自己启动之后发送到主题的消息
         * earliest：第一次从头开始消费，以后按照消息offset记录继续消费，这个需要区别于consumer.seekToBeginning(每次都从头开始消费)
         */
//        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 1000); // consumer给broker发送心跳的时间间隔
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 10 * 1000); // kafka如果超过10秒没有收到消费者的心跳，则会把消费者踢出消费组，进行rebalance，把分区分配给其他消费者
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500); // 一次poll最大拉取消息的条数，可以根据消费速度的快慢来设置
        props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 30 * 1000); // 如果两次poll的时间如果超出了30s的时间间隔，kafka会认为其消费能力过若，将其踢出消费组，将分区分配给其他消费者
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList(TOPIC_NAME));
        while (true) {
            /**
             * poll() API 是拉取消息的长轮询
             */
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d, key = %s, value = %s%n", record.partition(), record.offset(), record.key(), record.value());
            }
        }
        if (records.count() > 0) {
		    // 手动同步提交offset，当前线程会阻塞直到offset提交成功
		    // 一般使用同步提交，因为提交之后一般也没有什么逻辑代码了
		    consumer.commitSync();
		}	
    }
}

```

### 2.1.自动提交offset

设置自动提交参数 - 默认

```java
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true"); // 是否自动提交offset，默认true
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000"); // 自动提交offset的间隔时间
```

消费者poll到消息后默认情况下，会自动向broker的_consumer_offsets主题提交当前主题-分区消费的偏移量。

**自动提交会丢消息**：因为如果消费者还没消费完poll下来的消息就自动提交了偏移量，那么此时消费者挂了，于是下一个消费者会从已提交的offset的下一个位置开始消费。之前未被消费的消息就丢失了。

### 2.2.手动提交offset

- 设置手动提交参数

```java
props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
```

- 在消费完消息后进行手动提交

1. 手动同步提交

```java
if (records.count() > 0) {
    // 手动同步提交offset，当前线程会阻塞直到offset提交成功
    // 一般使用同步提交，因为提交之后一般也没有什么逻辑代码了
    consumer.commitSync();
}
```

2. 手动异步提交

```java
if (records.count() > 0) {
    // 手动异步提交offset，当前线程提交offset不会阻塞，可以继续处理后面的程序逻辑
    consumer.commitAsync(new OffsetCommitCallback()) {
        @Override
        public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
            if (exception != null) {
                System.err.println("Commit failed for " + offsets);
                System.err.println("Commit failed exception： " + exception.getStackTrace());
            }
        }
    }
}
```

### 2.3.长轮询poll消息

- 默认情况下，消费者一次会poll500条消息

```java
props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500); // 一次poll最大拉取消息的条数，可以根据消费速度的快慢来设置
```

- 代码中设置了长轮序的时间是1000毫秒

```java
            /**
             * poll() API 是拉取消息的长轮询
             */
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("收到消息：partition = %d,offset = %d, key = %s, value = %s%n", record.partition(), record.offset(), record.key(), record.value());
            }
```

意味着：

- 如果一次poll到了500条，就直接执行for循环
- 如果这一次没有poll到500条，且时间在1秒内，那么长轮询继续poll，要么到500条，要么到1s
- 如果多次poll都没达到500条，且1秒时间到了，那么直接执行for循环

如果两次poll的间隔超过30s，集群会认为该消费者的消费能力过弱，该消费者被踢出消费组，触发rebalance机制，rebalance机制会造成性能开销，可以通过设置这个参数，让一次poll的消息条数少一点。

### 2.4.消费者的健康状态检查

消费者每隔1s向kafka集群发送心跳，集群发现如果有超过10s没有续约的消费者，将被踢出消费组，触发该消费者的rebalance机制，将该分区交给消费组里的其他消费者进行消费。

### 2.5.指定分区消费

```java
consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
```

### 2.6.消息回溯消费

```java
consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
consumer.seekToBeginning(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
```

### 2.7.指定offset消费

```java
.consumer.assign(Arrays.asList(new TopicPartition(TOPIC_NAME, 0)));
consumer.seek(new TopicPartition(TOPIC_NAME, 0), 10);
```

### 2.8.从指定时间点开始消费

```java
List<PartitionInfo> topicPartitions = consumer.partitionsFor(TOPIC_NAME);
// 从1小时前开始消费
long fetchDataTime = new Date().getTime() - 1000 * 60 * 60;
Map<TopicPartition, Long> map = new HashMap();
for (PartitionInfo par : topicPartitions) {
    map.put(new TopicPartition(TOPIC_NAME, par.partition()), getchDataTime);
}
Map<TopicPartition, OffsetAndTimestamp> parMap = consumer.offsetsForTimes(map);
for (Map.Entry<TopicPartition, OffsetAndTimeStamp> entry : parMap.entrySet()) {
    TopicPartition key = entry.getValue();
    if (key == null || value == null) continue;
    Long offset = value.offset();
    System.out.println("partition-" + key.partition() + "|offset-" + offset);
    // 根据消费里的timestamp确定offset
    if (value != null) {
        consumer.assign(Arrays.asList(key));
        consumer,seek(key, offset);
    }
}
```

