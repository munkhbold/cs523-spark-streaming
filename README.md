This project is a demo of building a Proof-of-concept for Kafka + Spark streaming.

When considering this POC, I choose Twitter which is great source of streamed data, plus it would be easy to peform simple transformations on the data. So for this example, we will 
* create a stream of tweets that will be sent to a Kafka queue
* pull the tweets from the Kafka cluster
* calculate the character count and word count for each tweet
* save this data to a Hive table

---

## Requirements
- VirtualBox(Ubunty-19.04)
- Kafka
- Hadoop
- Hive
- Spark


## Installation

```bash
~$ pip3 install tweepy
~$ pip3 install pyspark
```


## Run the demo project

* ### Terminal 1 -- Zookeeper (Kafka cluster)

This is required for running a Kafka cluster.

```bash
~/kafka$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

* ### Terminal 2 -- Zookeeper (Kafka broker)

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

* ### Terminal 4 - Hadoop

```bash
~$ hdfs namenode -format
~$ start-dfs.sh
```

* ### Terminal 5 -- Hive Metastore

```
~/$ hive --service metastore
```

* ### Terminal 6 -- Hive

Make the database for storing our Twitter data:

``` bash
hive> CREATE TABLE tweets (text STRING, words INT, length INT)
    > ROW FORMAT DELIMITED FIELDS TERMINATED BY '\\|'
    > STORED AS TEXTFILE;
```

You can use `SHOW TABLES;` to double check that the table was created.

In order to make sure the transformed data is being stored in Hive, we need to enter the Hive shell so that we can query the tables

```bash
~$ hive

 ...

hive> use default;
hive> select count(*) from tweets;
```

* ### Terminal 7 -- Stream Producer

In this new terminal, we are going to run the stream producer

```~$ python3 tweet_stream.py```

* ### Terminal 8 -- Stream Consumer + Spark Transformer

Now we are ready to run the consumer. You can download `spark-streaming-kafka-0-8-assembly_2.11-2.4.7.jar` from [search.maven.org](https://search.maven.org/search?q=a:spark-streaming-kafka-0-8-assembly_2.11).
```
~$ spark-submit --jars spark-streaming-kafka-0-8-assembly_2.11-2.4.7.jar transformer.py
```
