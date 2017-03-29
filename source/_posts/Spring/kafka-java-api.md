---
categories: Spring
date: 2017-03-06 22:53
status: public
title: 'kafka Java API 使用'
---

## 一、添加 maven 依赖
***
**新版本的``kafka``客户端接口只需要这些包**
```xml
<!-- kafka jar -->
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>0.10.2.0</version>
</dependency>
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>0.10.2.0</version>
</dependency>
```

## 二、编写 Producer 客户端代码
***
```java
package org.demo.kafka;

import java.util.Properties;

import org.apache.kafka.clients.producer.*;

/**
 * Created by jzchen on 2017/3/4 0004.
 */
public class MyProducer {

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

        Producer<String, String> producer = new KafkaProducer<String, String>(props);
        for(int i = 0; i < 100; i++)
            producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)),
                new Callback() {
                    public void onCompletion(RecordMetadata metadata, Exception e) {
                        if(e != null)
                            e.printStackTrace();
                        System.out.println("The offset of the record we just sent is: " + metadata.offset());
                    }
                 }
            );

        producer.close();


    }
}
```

## 三、编写 Consumer 客户端代码
***
### 1、使用 hight level api 
```java
package org.demo.kafka;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Properties;

/**
 * Created by jzchen on 2017/3/5 0005.
 */
public class MyConsumer {


    public static void main(String[] args) {
        /**
         * 策略：自动提交，间隔1000ms
         * "at-most once delivery" guarantees
         * 劣势：在数据读入内存，consumer提交后程序失败，数据未持久化，丢失信息
         */
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList("my-topic"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records)
                System.out.printf("offset = %d, key = %s, value = %s", record.offset(), record.key(), record.value());
        }
    }
}
```

**使用 ``consumer.commitSync();``  手动 commit，标识records**
```java
package org.demo.kafka;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndMetadata;
import org.apache.kafka.common.TopicPartition;

import java.util.*;

/**
 * Created by jzchen on 2017/3/5 0005.
 */

public class MyConsumer {
    public static void main(String[] args) {
/**
         * 策略：手动提交，
         * "at-least once delivery" guarantees
         * 劣势：在数据读入内存并持久化，consumer提交前程序失败，下次将从之前的offset读取数据，造成数据重复
         */
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "false");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<String, String>(props);
        consumer.subscribe(Arrays.asList("my-topic"));
        final int minBatchSize = 200;
        List<ConsumerRecord<String, String>> buffer = new ArrayList<ConsumerRecord<String, String>>();
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                buffer.add(record);
            }
            if (buffer.size() >= minBatchSize) {
//                insertIntoDb(buffer);
                consumer.commitSync();  //手动提交，标识records
                buffer.clear();
            }
        }
    }
```

### 2、使用 low  level api 操作 offset

```java
TopicPartition topicPartition = new TopicPartition("my-topic",0);
        consumer.assign(Arrays.asList(topicPartition));
        consumer.seekToBeginning(Arrays.asList(topicPartition));
        try {
            while(true) {
                ConsumerRecords<String, String> records = consumer.poll(Long.MAX_VALUE);
                for (TopicPartition partition : records.partitions()) {
                    List<ConsumerRecord<String, String>> partitionRecords = records.records(partition);
                    for (ConsumerRecord<String, String> record : partitionRecords) {
                        System.out.println(record.offset() + ": " + record.value());
                    }
                    long lastOffset = partitionRecords.get(partitionRecords.size() - 1).offset();
                    consumer.commitSync(Collections.singletonMap(partition, new OffsetAndMetadata(lastOffset + 1)));
                }
            }
        } finally {
            consumer.close();
        }
    }
```