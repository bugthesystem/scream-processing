# scream-processing
Playground for Kafka Flink (CEP&amp; ML) Elasticsearch Kibana in Scala

# Contents
 - [Tech / Tools](#tech-tools)
 - [Env Setup (Docker Compose)](#)
 - [Env Setup (Kubernetes)](#)

# Tech / Tools
- [Scala](https://www.scala-lang.org/)
- [Sbt](http://www.scala-sbt.org/)
- [Kafka](https://kafka.apache.org/)
- [Flink](https://flink.apache.org/)
  - [FlinkCEP - Complex event processing for Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.2/dev/libs/cep.html)
  - [FlinkML - FlinkML - Machine Learning for Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.2/dev/libs/ml/index.html)
- [Elasticsearch](https://www.elastic.co/products/elasticsearch)
- [Kibana](https://www.elastic.co/products/kibana)
- [Docker](https://www.docker.com/)
- [Kubernetes](https://kubernetes.io/)

**_Kubernetes setup will be explained later on_**



# Env Setup (Docker Compose)
## Install Kafka
```sh
# TODO: convert to `docker-compose`
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
wget http://mirror.netinch.com/pub/apache/flink/flink-1.3.0/flink-1.3.0-bin-hadoop27-scala_2.11.tgz

tar xzf flink-*.tgz   # Unpack the downloaded archive
cd flink-1.3.0

# Start Flink
./bin/start-local.sh

#OR
docker pull flink

docker run -t -p 8081:8081 flink local
```

## Install ES/Kibana
```sh
# I don't need to Logstash but we can use `docker-elk` for playground
# ES/Kibana docker-compose and kubernetes setup will be provided

git clone git@github.com:deviantony/docker-elk.git
cd docker-elk
docker-compose up
```

# Env Setup (Kubernetes)
```sh
# TODO
```

# Project Setup

## Setup Project /w sbt
```sh
# Run following script ot clone starter repo
bash <(curl https://flink.apache.org/q/sbt-quickstart.sh)

#or
git clone https://github.com/tillrohrmann/flink-project.git your-project-name-here
```

## Project structure and dependencies
```scala
 // `flink-scala` and `flink-streaming-scala` will be pre-configured
"org.apache.flink" %% "flink-scala" % flinkVersion % "provided",
"org.apache.flink" %% "flink-streaming-scala" % flinkVersion % "provided",

 // Add `flink-connector-kafka` to dependencies
 "org.apache.flink" % "flink-connector-kafka-0.10_2.10" % "1.3.0"
```
