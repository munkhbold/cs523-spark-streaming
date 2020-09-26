This project is a demo of building a Proof-of-concept for Kafka + Spark streaming.

When considering this POC, I choose Twitter which is great source of streamed data, plus it would be easy to peform simple transformations on the data. So for this example, we will 
* create a stream of tweets that will be sent to a Kafka queue
* pull the tweets from the Kafka cluster
* calculate the character count and word count for each tweet
* save this data to a Hive table

To do this, we are going to set up an environment that includes 
* a single-node Kafka cluster
* a single-node Hadoop cluster
* Hive and Spark

---

## Requirements
- VirtualBox(Ubunty-19.04)
- Kafka
- Hadoop
- Hive
- Spark


## Installation
`pip3 install tweepy` 


## Run the demo project

* ### Terminal 1 -- Zookeeper

This is required for running a Kafka cluster.

```bash
~/kafka$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

* ### Terminal 2 -- Zookeeper

Now we need to start the Kafka broker.

```bash
~/kafka$ bin/kafka-server-start.sh config/server.properties
```

* ### Terminal 3 -- Kafka Config
Now that we have our Kafka cluster running, we need to create the topic that will hold our tweets.

First check if any topics exist on the broker (this should return nothing if you just started the server)

```bash
~/kafka$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

Now make a new topic named `tweets`

```bash
~/kafka$ bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic tweets
```

* ### Terminal 4 -- Hive Metastore

hive --services metastore

* ### Terminal 5 -- Hive

In order to make sure the transformed data is being stored in Hive, we need to enter the Hive shell so that we can query the tables

```
~$ hive

 ...

hive> use default;
hive> select count(*) from tweets;
```

* ### Terminal 6 -- Stream Producer

In this new terminal, we are going to run the stream producer

```~$ python3 tweet_stream.py```

* ### Terminal 7 -- Stream Consumer + Spark Transformer

Now we are ready to run the consumer. You can download `spark-streaming-kafka-0-8-assembly_2.11-2.4.7.jar` from [search.maven.org](https://search.maven.org/search?q=a:spark-streaming-kafka-0-8-assembly_2.11).
```
~$ spark-submit --jars spark-streaming-kafka-0-8-assembly_2.11-2.4.3.jar transformer.py
```
