# Apache Kafka Theory
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
_( Kafka Message (a) Serialization (b) Deserialization on Producer and Consumer nodes, respectively; by [Rahul Siyanwal](https://github.com/rsiyanwal))_

- **Consumers:** Kafka Consumers read data from a topic using the Pull method. Consumers can read data from one or more topics. Consumers can even read data from a specific partition of a topic. Consumers already know from which broker they have to pull the data. Data is read from low to high offset within each partition. Kafka Consumers have Deserializers that transform bytes into a data object. Deserializers are used on the value and key of the data. The serialization and deserialization type must not be changed during a topic lifecycle.

- **Brokers:** A Kafka Cluster is an ensemble of multiple Kafka Brokers, and a broker is just a server. An integer ID identifies a Kafka Broker. Each broker contains only specific topic partitions, which means the data is distributed across all brokers. We get connected to the entire cluster once we connect to any broker. Generally, it is recommended to get started with three brokers and scale it as required. 
![Kafka-Cluster drawio](https://user-images.githubusercontent.com/11557572/196426948-7867ec40-6735-4172-8a2c-ade10041e9e0.png)<br/>
_( Two topics distributed between a Kafka Cluster of three brokers ; by [Rahul Siyanwal](https://github.com/rsiyanwal))_ <br/>
Let's take the example of three brokers with two different topics. Topic-A has 3 partitions, and Topic-B has 2 partitions. Total 5 partitions of two Topics will be spread across 3 brokers. This is called Horizontal Scaling. Each Kafka Broker in a cluster is called a "Bootstrap Server." In a cluster, we only need to connect to one broker, and the clients will know how to be connected with the entire cluster. Kafka Client connects to one broker, and that broker sends the list of all the brokers in the cluster and more data (which broker has which partition). With the list provided by a broker, the client will be able to connect to the broker it needs to produce or consume the data.

- **Consumer Group:** All the Kafka Consumers read data as a Consumer Group. Each consumer in a consumer group reads from an exclusive partition. If, in a consumer group, we have more consumers than a partition of a topic, then extra consumers would remain inactive. We can have more than one consumer group. Kafka saves the offset at which a consumer group is reading. A consumer in a consumer group should be periodically committing offsets. By committing the offset, we tell Kafka Broker how far we have processed the data. If a consumer dies and returns, it starts reading the data from where it was committed. By default, consumers in Java API automatically commit offsets (at least once). We have three different settings to commit the offset. They are: 
  - **At Least Once (Usually Default):** Offsets are committed only after the message is processed. If the processing goes wrong, the message will be read again. This setting is practical when duplicate data won't impact our application. 
  - **At Most Once:** The offset is committed as soon as the messages are received. If the processing goes wrong, then some of the messages will be lost and won't be available to be read again
  - **Exactly Once:** When we read from a topic and write to a topic, we use the Transactional API. This offset setting is used for Kafka-to-Kafka workflow.

- **Topic Replication:** Topics must have a replication factor greater than 1 (usually 2 or 3). If the Kafka Broker is down, another broker can serve the data to the client. For Example, Let's have Topic - A with 2 partitions and a replication factor of 2. We have three brokers, Broker 101, Broker 102, and Broker 103 (figure 6.21). If Broker 102 stops working, then Broker 101 and Broker 103 can still service the data to Kafka Client. Since we have Replicas, we now need a leader of the partition. The leader is the Broker for a particular partition. There can only be one leader for a partition at a time. Producers and Consumers only interact with the leader of the partition. In the diagram below, Broker 101 is the leader of Partition 0, and Broker 102 is the leader of Partition 1. The data replication is updated according to the data received and read. When the replicas are in synchronization, they are called In-Sync Replica (ISR).
![Kafka-Replication drawio](https://user-images.githubusercontent.com/11557572/196468480-3ae24840-17d7-4c5f-8bb0-ae0b32144390.png)<br/>
_( Topic Replication and Partition Leaders ; by [Rahul Siyanwal](https://github.com/rsiyanwal))_ <br/>

- **Zookeeper:** Zookeeper is the tool that manages all the brokers (and keeps a list of them). Zookeeper helps elect the leader for partitions of topics because whenever a broker is down, we need to choose a new leader for the partition. Also, Zookeeper sends notifications to all brokers in case of recent topics, topic deletion, Broker dies, the Broker returns, etc. Up to Kafka version 2.X we have to use Zookeeper for all these tasks. However, newer versions of Kafka are slowly replacing Zookeeper with KafkaRaft. In 4.X versions of Kafka, we won't have Zookeeper. Zookeeper operates with an odd number of servers (1, 3, 5, 7). Zookeeper only has one leader server, and the rest of the servers are followers. To manage Kafka brokers, we must be familiar with Zookeeper. 

# Apache Kafka Hands-on
## Requirements and Installations
### Installing Kafka on Windows
Please perform the following steps:
  1. **Installing WSL2:** Please make sure that you are using Windows 10 or above. We will use WSL2 on Windows for Kafka. To download, go Microsoft Store (You can find the app in Start menu) and search for "Ubuntu". For demonstration, I am using Ubuntu 20.04. Alternatively, use command ```wsl --install``` in Command Line or Powershell which install Ubuntu by default. 
  2. **Installing JDK version 11:** I am using Amazon Corretto 11. To download, please go to [Amazon Corretto 11 download page](https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/generic-linux-install.html) and follow the instructions. Alternatively, use the following commands: 
```
   wget -O- https://apt.corretto.aws/corretto.key | sudo apt-key add - 
   sudo add-apt-repository 'deb https://apt.corretto.aws stable main'
   sudo apt-get update; sudo apt-get install -y java-11-amazon-corretto-jdk
  ```
  3. 
