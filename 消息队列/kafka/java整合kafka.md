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

