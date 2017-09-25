---
categories: kafka
date: 2017-09-10 14:30
title: kafka Stream API（未完）
---



# KStream API

## filter

```Java
org.apache.kafka.streams.kstream.KStream

KStream<K, V> filter(Predicate<? super K, ? super V> predicate)
```

创建一个新的 KStream 包含所有满足 predicate 的记录，不满足 predicate 的所有记录将被丢弃。这是一个无状态 记录-到-记录 操作。



## flatMap

```java
org.apache.kafka.streams.kstream.KStream

<KR, VR> KStream<KR, VR> flatMap(KeyValueMapper<? super K, ? super V, ? extends Iterable<? extends KeyValue<? extends KR, ? extends VR>>> mapper)
```
 将每一个记录转为化0个或多个记录，(key 和 value 的类型都可以任意替换为其他类型)。KeyValueMapper 会应用到每一个输入记录并计算出0个或者多个记录，因此，输入的记录 <K,V> 可以被转化为输出记录 <K': V'>,<K'': V''>, ....这是一个无状态 记录-记录 操作(cf. transform(TransformerSupplier, String...) for 有状态记录转换器)
```java
//以下的例子将包含有句子的的输入记录 <null:String> 切分为单词，并为每一个单词发出一条 <word:1> 的记录。
KStream<byte[], String> inputStream = builder.stream("topic");
KStream<String, Integer> outputStream = inputStream.flatMap(new KeyValueMapper<byte[], String, Iterable<KeyValue<String, Integer>>> {
  Iterable<KeyValue<String, Integer>> apply(byte[] key, String value) {
    String[] tokens = value.split(" ");
    List<KeyValue<String, Integer>> result = new ArrayList<>(tokens.length);

    for(String token : tokens) {
      result.add(new KeyValue<>(token, 1));
    }

    return result;
  }
});
```



## flatMapValues

```java
org.apache.kafka.streams.kstream.KStream

<VR> KStream<K, VR> flatMapValues(ValueMapper<? super V, ? extends Iterable<? extends VR>> processor)
```
通过将输入中的每一个记录转化为0个或多个和原来相同(未修改过的) key 的 value 来创建一个新的输出（value 的类型可以替换为任意其他类型）。提供的 ValueMapper 将应用到每一个输入的记录计算出0个或多个输出 value。 因此，一个输入记录 <K, V> 可以被转化为 <K: V'>, <K: V''>, ....这是一个无状态 记录-记录 操作，(cf. transformValues(ValueTransformerSupplier, String...) for 有状态 value 转换器).

```java
// 以下的例子把包含有句子作为 value 的输入记录 <null:String>，切分为单词
The example below splits input records <null:String> containing sentences as values into their words.
  
KStream<byte[], String> inputStream = builder.stream("topic");
KStream<byte[], String> outputStream = inputStream.flatMapValues(new ValueMapper<String, Iterable<String>> {
  Iterable<String> apply(String value) {
    return Arrays.asList(value.split(" "));
  }
});
      
```



## map

```java
org.apache.kafka.streams.kstream.KStream

<KR, VR> KStream<KR, VR> map(KeyValueMapper<? super K, ? super V, ? extends KeyValue<? extends KR, ? extends VR>> mapper)
```
将输入中的每一个记录转化为输出的一个新的记录（key 和 value 的类型可以替换为任意其他类型）。提供的 KeyValueMapper 将应用到每一个输入的记录计算出一个新的记录。因此，一个输入记录 <K,V> 可以被转化为一个输出记录 <K': V'>。这是一个无状态 记录-记录 操作。(cf. transform(TransformerSupplier, String...) for stateful record transformation).

```java
 // 以下的例子把 String 类型的 key 格式化为“大写格式”的字母，并计算 value 字符串的单词个数
KStream<String, String> inputStream = builder.stream("topic");
KStream<Integer, String> outputStream = inputStream.map(new KeyValueMapper<String, String, KeyValue<String, Integer>> {
  KeyValue<String, Integer> apply(String key, String value) {
    return new KeyValue<>(key.toUpperCase(), value.split(" ").length);
  }
});
```



## mapValues

```java
org.apache.kafka.streams.kstream.KStream

<VR> KStream<K, VR> mapValues(ValueMapper<? super V, ? extends VR> mapper)
```

 将输入中的每一个记录的 value 转化为输出的一个新的 value (可能是一个新的类型)。提供的 ValueMapper 将应用到每一个输入记录的 value 并计算出一个新的 value。因此，一个输入记录 <K,V> 可以被转化为一个输出记录 <K: V'>。这是一个无状态 记录-记录 操作。(cf. transformValues(ValueTransformerSupplier, String...) for stateful value transformation).
```java
  // 下面的例子计算 value string 的 token 数量
KStream<String, String> inputStream = builder.stream("topic");
KStream<String, Integer> outputStream = inputStream.mapValues(new ValueMapper<String, Integer> {
  Integer apply(String value) {
    return value.split(" ").length;
  }
});
```



## groupByKey

```java
org.apache.kafka.streams.kstream.KStream
KGroupedStream<K, V> groupByKey()
```

-  根据记录当前的 key 将他们分组到一个 KGroupedStream 中同时保存原始的 values 和默认的序列化和反序列化器。根据记录的 key 将一个 stream 分组在进行聚合(aggregation)操作之前是必要的，它将应用数据中(cf. KGroupedStream)。如果记录的 key 为 null，它将不会包含在作为返回结果的 KGroupedStream。
-  如果一个变更 key 的操作在分组操作之前进行，【e.g., selectKey(KeyValueMapper), map(KeyValueMapper), flatMap(KeyValueMapper), or transform(TransformerSupplier, String...)】,并且没有数据重分配(redistribution)在这之后发生(e.g., via through(String)) ，那么一个内部的重分区主题(repartitioning topic) 将被创建。这个主题将被命名为 "${applicationId}-XXX-repartition"，applicationId 是用户在 StreamsConfig 中通过 APPLICATION_ID_CONFIG 参数指定的，"XXX" 是一个内部生成的名字，"-repartition"是一个固定的后缀，你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。
-  在这些情形下，这个 stream 中所有数据将被重分配到重分区主题，通过将所有记录重新写入的方式，并且从该 stream 中重新读取所有的记录，这样作为结果返回的 KGroupedStream 就已经基于新的 key 正确地重新分区。如果最后的变更 key 的操作改变了 key 的类型，它将重新调用 groupByKey(Serde, Serde) 来代替这个分组操作。

**Note**： 在分组之前有变更 key 的操作，那么分组操作将产生一个内部主题并重新分。



## groupBy

```java
org.apache.kafka.streams.kstream.KStream
<KR> KGroupedStream<KR, V> groupBy(KeyValueMapper<? super K, ? super V, KR> selector)
```

- 使用了提供的 KeyValueMapper 和默认的序列化和反序列化器重新选择(select) 的 key 来把记录进行分组。根据记录的 key 将一个 stream 分组在进行聚合(aggregation)操作之前是必要的，它将应用数据中(cf. KGroupedStream)。KeyValueMapper 选择了一个新的 key（需要相同的类型）同时保存了原始的 values，如果新的 key 为 null，它将不会包含在作为返回结果的 KGroupedStream。
- 因为选择了一个新的 key，一个内部的重分区主题将被创建。这个主题将被命名为 "${applicationId}-XXX-repartition"，applicationId 是用户在 StreamsConfig 中通过 APPLICATION_ID_CONFIG 参数指定的，"XXX" 是一个内部生成的名字，"-repartition"是一个固定的后缀，你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。
- 这个 stream 中所有数据将被重分配到重分区主题，通过将所有记录重新写入的方式，并且从该 stream 中重新读取所有的记录，这样作为结果返回的 KGroupedStream 就已经基于新的 key 正确地重新分区。如果最后的变更 key 的操作改变了 key 的类型，它将重新调用 groupByKey(Serde, Serde) 来代替这个分组操作。
- 这个操作等效于在调用 selectKey(KeyValueMapper) 之后调用 groupByKey()。如果 key 的类型改变了，它将重新调用 groupByKey(Serde, Serde) 来代替这个分组操作。




## join

```Java
org.apache.kafka.streams.kstream.KStream

<VO, VR> KStream<K, VR> join(
  	KStream<K, VO> otherStream,
	ValueJoiner<? super V, ? super VO, ? extends VR> joiner,
	JoinWindows windows)
```
- 使用 windowed 内连接，连接当前 KStream 和另外一个 KStream 的所有记录，使用默认的序列化房序列化器。连接根据记录的 key 是否符合连接属性 `hisKStream.key == otherKStream.key` 。此外，只有两边的记录的时间戳差距在  JoinWindows 定义的时间

  内，他们才能被连接。

- 对于每一对连接的记录，都会调用提供的 ValueJoiner 来计算出可以是任意类型的结果。如果输入记录的 key 或 value 为 null，该记录将不会为包括在连接操作中，因此没有记录会被添加到结果 KStream 中。

Example (假设所有的输入记录都在窗口内):

| this    | other   | result                 |
| ------- | ------- | ---------------------- |
| <K1: A> |         |                        |
| <K2: B> | <K2: b> | <K2: ValueJoiner(B,b)> |
|         | <K3: c> |                        |

- 连接的输入流（更准确地说是他们的下游 topics）需要有相同数量的分区。如果不是这种情况，你需要在连接之前调用 `through(String)` （为其中一个 stream），使用一个预先创建的有正确分区数的主题。并且，每个输入 stream 需要协同 key 的分区策略（例如：使用相同的 partitioner）。如果不能满足这个要求，Kafka Streams 将自动重新分配数据，例如，将创建一个内部的重分配 topic ，将数据写入并在真正连接之前再次从这个 topic 读取数据。重新分配的 topic 会被命名为 "${applicationId}-XXX-repartition", "applicationId" 为用户在 StreamsConfig `APPLICATION_ID_CONFIG`指定的名称， "XXX"是一个内部生成的名称，"-repartition"是一个固定的后缀，你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。
- 重分配可能发生在一个或多个连接的 KStreams。这种情况下，stream 所有的数据都将被重定向写入到重新分配的 topic 中，并重新读取，这样连接的 KStream 就能在 key 上正确地分区。
- 连接的 KStream 都将被保存在名称自动生成的本地 state stores 。为了失效备份和恢复，每个存储都将备份到一个Kafka创建的一个内部 changelog topic 。命名为 "${applicationId}-storeName-changelog"，"applicationId" 为用户在 StreamsConfig `APPLICATION_ID_CONFIG`指定的名称， "XXX"是一个内部生成的名称，"-changelog"是一个固定的后缀，你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。



其他连接类似

- leftJoin(KStream, ValueJoiner, JoinWindows)

- outerJoin(KStream, ValueJoiner, JoinWindows)

- leftJoin(KTable, ValueJoiner)

- join(GlobalKTable, KeyValueMapper, ValueJoiner)

  …….

  ​

## selectKey

```java
org.apache.kafka.streams.kstream.KStream

<KR> KStream<KR, V> selectKey(KeyValueMapper<? super K, ? super V, ? extends KR> mapper)

```

- 为每一条记录设置一个新的 key（可能是新类型）。提供的 KeyValueMapper 将被应用到每一条记录上并计算出一个新的 key。因此，一个输入记录 <K,V> 可以被转换为输出记录 <K': V>。这是一个无状态 记录-到-记录 操作。
- 例如，你可以使用这中转换，在 KeyValueMapper 中为没有 key 的输入记录 <null,V> 提取出一个 key。

 The example below computes the new key as the length of the value string.

```java
  KStream<Byte[], String> keyLessStream = builder.stream("key-less-topic");
  KStream<Integer, String> keyedStream = keyLessStream.selectKey(new KeyValueMapper<Byte[], String, Integer> {
      Integer apply(Byte[] key, String value) {
          return value.length();
      }
  });
```

如果一个基于 key 的操作（如：aggregation 或 join）应用于结果 KStream，那么像这样设置一个新的 key 可能会造成内部的数据重分配。





# KGroupedStream

## reduce


```Java
org.apache.kafka.streams.kstream.KGroupedStream
KTable<K, V> reduce(Reducer<V> reducer, String queryableStoreName)
```
- 根据分组后的 key 合并这个 Stream 中记录的 values。null key 或者 value 的记录会被忽略。合并意味着集合结果的类型和输入的 value 是一致的(c.f. aggregate(Initializer, Aggregator, Serde, String))。合并的结果会被写进一个本地的 KeyValueStore(基本上是一个持续更新的持久化视图  (which is basically an ever-updating materialized view) )，它能通过 queryableStoreName 查询。更详细地，store 的更新会被发送到下游进入一个 KTable changelog stream。
- 指定的 Reducer 被应用到每个一个输入记录，并使用当前聚合值和记录的 value 计算出一个新的聚合值。如果没有当前聚合值，Reducer将不会被使用并且新的聚合值将直接使用记录的 value。因此，reduce(Reducer, String) 可以用来作为聚合函数计算，如：sum, min, or max。
- 不是所有的更新都能发送到下游，因为一个内部的 cache 被运用来去除重复的、持续更新的相同 key的记录。更新增殖的速率基于你输入数据的速率，唯一 key 的数量，并行运行的 Kafka Streams 实例的数量和配置参数配置的 cache 大小和提交的间隔。


- 想要查询本地 KeyValueStore 需要通过 KafkaStreams#store(...) 获得：

```java
KafkaStreams streams = ... // compute sum
ReadOnlyKeyValueStore<String,Long> localStore = streams.store(queryableStoreName, QueryableStoreTypes.<String, Long>keyValueStore());
String key = "some-key";
Long sumForKey = localStore.get(key); // key must be local (application state is shared over all running Kafka Streams instances)
```

- 对于非本地的 keys，需要实现客户化的 RPC 机制并使用 **KafkaStreams.allMetadata()** 在并行运行的 Kafka Streams 应用实例中查询你需要的 key 的 value。


- 为了失效备份和恢复，store 通过在 Kafka 中创建一个内部的 changelog topic 来备份。因此，store 的名字必须是一个有效的 kafka 主题名并且不能包含除`'.', '_' and '-'` 之外的 ASCII 字符。changelog topic 将被命名为 `"${applicationId}-${queryableStoreName}-changelog"`,  "applicationId" 是用户在 StreamsConfig 中通过 `APPLICATION_ID_CONFIG` 参数指定的，"queryableStoreName" 用户是提供的 queryableStoreName，而 "-changelog" 是固定的后缀。你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。

Note：首先产生 store，产生下游的 KTable changelog stream，并产生备份的 topic



## aggregate

```java
org.apache.kafka.streams.kstream.KGroupedStream

<W extends Window, VR> KTable<Windowed<K>, VR> aggregate(Initializer<VR> initializer,
	Aggregator<? super K, ? super V, VR> 
	aggregator,
	Windows<W> windows,
	Serde<VR> aggValueSerde,
	String queryableStoreName)
```
- 通过分组后的 key 和 定义的 windows 聚合记录中的 value。key 或 value 为 null 的记录将被忽略。聚集是一个泛型化的结合，因为他是通过 `reduce(...)` 实现的。例如，允许结果拥有和输入不同的类型。指定的窗口既可以是 hopping time windows 也可以是 tumbling(c.f. TimeWindows) 或者自己定义的 landmark windows (c.f. UnlimitedWindows)。聚合的结果被写进本地的 windowed KeyValueStore (which is basically an ever-updating materialized view)，它可以通过提供的 queryableStoreName 查询到。窗口将一直保留直到他们的保留期超时(c.f. Windows.until(long))。并且，向 store 的进行的更新，将被发送到下游stream进入一个 windowed KTable changelog stream。“windowed” 意味着 KTable的 key 是一个普通 key 和 Window ID 的组合 key。


- 指定的 Initializer 将在第一个输入记录被处理之前，直接应用到每个窗口，以提供一个初始的聚合值来处理第一条记录。指定的 Aggregator 将被应用到每一条输入记录，并根据当前的聚合值（或者 Initializer 提供的初始聚合值）和当前的记录值计算出一个新的聚合值。因此，这个方法能用来作为聚合函数如 count (c.f. count(String))。
- 不是所有的更新都会被发送到下游，因为一个内部 cache 被用来给持续发送的、相同的 window 和 key 的记录去重。记录增加的速率取决于你输入数据的速率，不同 key 的数量，并行运行的 Kafka Streams 实例数量，和设置的 cache 大小以及提交的间隔。
- To query the local windowed KeyValueStore it must be obtained via KafkaStreams#store(...):

```java
KafkaStreams streams = ... // some windowed aggregation on value type double
ReadOnlyWindowStore<String,Long> localWindowStore = streams.store(
  	queryableStoreName, 
  	QueryableStoreTypes.<String, Long>windowStore());
String key = "some-key";
long fromTime = ...;
long toTime = ...;
WindowStoreIterator<Long> aggForKeyForWindows = localWindowStore.fetch(key, timeFrom, timeTo); // key must be local (application state is shared over all running Kafka Streams instances)
```

- 如果需要查询费本地的 key，你必须实现一个客户化的 RPC 机制，然后通过 `KafkaStreams.allMetadata()`  来从你的 Kafka Streams 应用程序中并行运行的实例

  中查询 value。

- 为了失效备份和恢复，store 通过在 Kafka 中创建一个内部的 changelog topic 来备份。因此，store 的名字必须是一个有效的 kafka 主题名并且不能包含除`'.', '_' and '-'` 之外的 ASCII 字符。changelog topic 将被命名为 `"${applicationId}-${queryableStoreName}-changelog"`,  "applicationId" 是用户在 StreamsConfig 中通过 `APPLICATION_ID_CONFIG` 参数指定的，"queryableStoreName" 用户是提供的 queryableStoreName，而 "-changelog" 是固定的后缀。你可以通过 KafkaStreams.toString() 取得所有生成的内部主题名称。





class KStreamMap<K, V, K1, V1> implements ProcessorSupplier<K, V> 

class KStreamMapValues<K, V, V1> implements ProcessorSupplier<K, V> 



# KTable API

Note: KTable 没有 groupByKey 方法，因为KTable 包含了整个数据集，从 Topic 读取数据的时候，会去除相同 key 的重复记录，只保留最新的记录。因此，每个 key 只会有一条记录，那么 groupByKey 就没有意义了。

# KGroupedTable API

# StateStore

