# scream-processing
 Playground for Kafka Flink (CEP&amp; ML) Elasticsearch Kibana in Scala

## Install Kafka
```sh
wget http://mirror.netinch.com/pub/apache/kafka/0.10.2.0/kafka_2.11-0.10.2.0.tgz

tar -xzf kafka_2.11-0.10.2.0.tgz

cd kafka_2.11-0.10.2.0

#  Start Zookeeper
bin/zookeeper-server-start.sh config/zookeeper.properties

# Start Kafka server
bin/kafka-server-start.sh config/server.properties

# Create topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic scilink

# 
# To send message
#
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic scilink

# To consume message
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic scilink --from-beginning
```

## Install Flink
```sh
#TODO
```

## Install ES/Kibana
```sh
#TODO
```

## Setup Project /w sbt
```sh
#TODO
```
