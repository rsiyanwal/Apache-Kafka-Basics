# Apache Kafka Basics
Apache Kafka is also a messaging protocol. Just like MQTT, it also works on Pub/Sub model. The difference is that we use MQTT when the connection is unreliable, and we have to send data between thousands of clients. MQTT is a lightweight push-based messaging solution. Apache Kafka is an event-streaming platform used for data processing and integration. Kafka provides high performance (latency of less than 10ms), which means it is almost real-time. 

## Publish-Subscribe Model
In the Publish-Subscribe model, we have three entities: **Publisher**, **Subscriber**, and **Broker**.
- Publishers are those entities that produce data (could be sensors). In Kafka, they are called **Producers**.
- Subscribers are those entities that consume the data for some work to do on their part. In Kafka, they are called **Consumers**.
- Brokers are data managers.

A publisher "pushes" the data into the broker and the subscriber "gets" the data from the broker. Subscribers subscribe to various" topics" according to their needs, and brokers provide the topic's data to these subscribers. The publishers and subscribers are not connected; they are unaware of each other. We need the Publish-Subscribe model because it solves a huge issue in IoT deployment. Suppose we have ten sensors that send data to 10 different raspberry pi devices. If all the sensors send data to all the raspberry pi devices, then we would have 100 connections. Now, suppose the number of sensors and raspberry pi devices is even more significant. Ten thousand sensors send data to ten thousand raspberry pi devices, which makes 100 million connections, a considerable number hard to maintain. With the Publish-Subscribe model, the sensors can publish data to topic A, the subscribers can consume the data from topic A, and the broker handles all the requests. If there are ten thousand sensors sending data to the topic and ten thousand raspberry pi devices consuming the data from topic A, then we need only twenty thousand connections.

![Normal Connections vs. Pub-Sub Model](https://user-images.githubusercontent.com/11557572/196355586-4e4d3c15-4930-40b1-a8b9-cf2fcf7c7668.png)
 _( by [Rahul Siyanwal](https://github.com/rsiyanwal))_
 
 ## Kafka Building Blocks
 Kafka have five building blocks: **Topics**, **Partitions**, **Producer**, **Consumer**, and **Broker**, 
 - **Topics:** Topics in Kafka are a specific data stream comparable to ”Table” in the database but without constraints. We can send anything we want in Kafka Topic without data verification. Also, we can have as many topics as we wish. A way to identify a topic in Kafka is by its name. For example, we can have topics such as logs, purchases, twitter-tweets, city, school, etc. We can send any message format to Kafka. Kafka topics are immutable, which means that once data is written to a partition, it can’t be changed. Data in Kafka is kept for a limited time only. The default time for a message is one week, but it is configurable.
 - **Partitions:** We can split Kafka Topics into partitions. Messages sent to the Kafka topic will end up in these partitions, and messages in each partition will be in order. The first ID of a message at the beginning of a partition will start with 0 and increment as more data comes. The ID of a message in a partition is called Kafka Partition Offset.
![Kafka-Partition drawio](https://user-images.githubusercontent.com/11557572/196405449-5a0893e3-82b8-4c2a-9898-03b197200af4.png)
_( Kafka Topic Partitions; by [Rahul Siyanwal](https://github.com/rsiyanwal))_ <br/>Offset only have some meaning for a specific partition because, for example, offset 5 of partition 3 does not represent the same data as offset 5 of partition 1. Order of messages is guaranteed within a partition and not across the partitions. Unless a "Key" is provided, the data is assigned to a partition randomly.
- **Producer:** Kafka Producers write data on a topic. Producers, additionally, can send a "key" with the data. The data is sent to partitions in Round-Robin Fashion if the key is null. If the key is not null, then all the messages for that key will always go to the same partition (hashing). Producers can choose to receive acknowledgment whenever they write the data. There are three settings for acknowledgment, which are as follows:
  - acks == 0: Producer will not wait for the acknowledgement and it could result in possible data loss
  - acks == 1: : Producer waits for the leader acknowledgement. In acks = 1 we have limited data loss because it might be possible that replicas didn’t receive the data because they are not in sync
  - acks == all: Producer will wait for the leader as well as all the replicas for the acknowledgement which result in no data loss.

- Kafka only accepts a series of bytes as input and a series of bytes as output. Therefore, we need a serializer to transform our data object into bytes. Message Serialization is used in both value and key. Kafka comes with common Serializers that help to convert a data object (Including JSON, String, Int, Float, Avro, Protobuf, etc.) to byte. In default, the keys are hashed using **murmur2** algorithm.
![Kafka-Message-Serialization-1](https://user-images.githubusercontent.com/11557572/196374105-6f8a4c43-1379-4efd-9a0b-da2e8a0a3f64.png)<br/>
_( Kafka Message (a) Serialization (b) Deserialization on Publisher and Subscriber nodes; by [Rahul Siyanwal](https://github.com/rsiyanwal))_

- **Consumers:** Kafka Consumers read data from a topic using the Pull method. Consumers can read data from one or more topics. Consumers can even read data from a specific partition of a topic. Consumers already know from which broker they have to pull the data. Data is read from low to high offset within each partition. Kafka Consumers have Deserializers that transform bytes into a data object. Deserializers are used on the value and key of the data. The serialization and deserialization type must not be changed during a topic lifecycle.

