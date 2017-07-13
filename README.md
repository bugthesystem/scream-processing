# Scream processing (Patoz)
Playground for Apache Kafka, Apache Flink (CEP, ML) Elasticsearch and Kibana in Scala

```sh
 _ _ _ _ _ _ _ _                _ _ _ _           _ _ _ _ _ _ _         _ _ _ _ _ _ 
/         Akka   \             /        \        /               \     |x          |
|   Vert.x       |             | Flink  |    _ _ | Elasticsearch | --- |  Kibana   |
|                |             |        |   /    \ _ _ _ _ _ _ _ /     |_ _ _ _ _ _|
|       Node.js  | -- Kafka ---|   Job  | /       _ _ _ _ _ _ _   
|   Spring Boot  |             | Job    | \      /              \      _ _ _ _ _ _  
|      .NET Core |             |   Job  |   \ _ _|    Kafka     | -- / Other apps  \
\_ _ _ _ _ _ __ /              \ _ _ _ _/        \ _ _ _ _ _ _ _/    \ _ _ _ _ _ _ /

```

# Contents
 - [Tech / Tools](#tech--tools)
 - [Env Setup (Docker Compose)](#env-setup-docker-compose)
 - [Env Setup (Kubernetes)](#env-setup-kubernetes)

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
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic scream-processing

# 
# To send message
#
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic scream-processing

# To consume message
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic scream-processing --from-beginning
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
:warning: `Flink Elasticsearch connector` for `Elasticsearch 5` is missing in `Maven` repository atm.

```sh
#Elasticsearch 2.4.5
docker run --name scream-processing-elasticsearch -p 9200:9200 -p 9300:9300 \
           -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" -d elasticsearch:2.4.5

# Kibana 4.6.4
docker run --name scream-processing-kibana --link scream-processing-elasticsearch:elasticsearch -p 5601:5601 -d kibana:4.6.4
```

# Env Setup (Kubernetes)
```sh
# clone docker-flink/examples repo
git clone git@github.com:docker-flink/examples.git
 
cd docker-flink
 
# Build the Helm archive:
helm package helm/flink/
 
# Deploy a non-HA Flink cluster with a single taskmanager:
helm install --name my-cluster flink*.tgz
 
# Deploy a non-HA Flink cluster with three taskmanagers:
helm install --name my-cluster --set flink.num_taskmanagers=3 flink*.tgz
 
# Deploy an HA Flink cluster with three taskmanagers:
cat > values.yaml <<EOF
flink:
  num_taskmanagers: 3
  highavailability:
    enabled: true
    zookeeper_quorum: <zookeeper quorum string>
    state_s3_bucket: <s3 bucket>
    aws_access_key_id: <aws access key>
    aws_secret_access_key: <aws secret key>
EOF
 
# use modified values.yaml
helm install --name my-cluster --values values.yaml flink*.tgz
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
### SBT build file
```scala
 scalaVersion in ThisBuild := "2.11.7"

val flinkVersion = "1.3.0"

val flinkDependencies = Seq(
  "org.apache.flink" %% "flink-scala" % flinkVersion % "provided",
  "org.apache.flink" %% "flink-streaming-scala" % flinkVersion % "provided",
  "org.apache.flink" % "flink-connector-elasticsearch2_2.10" % flinkVersion,
  "org.apache.flink" % "flink-connector-kafka-0.10_2.10" % flinkVersion
)

// Joda time. (HACK)
assemblyMergeStrategy in assembly := {
  case PathList("org", "joda", "time", "base", "BaseDateTime.class") => MergeStrategy.first
  case x =>
    val oldStrategy = (assemblyMergeStrategy in assembly).value
    oldStrategy(x)
}
```

### Testing
_:warning: Kafka as datasource and Elasticsearch for output in my case_

- [x] Test Base using `LocalFlinkMiniCluster`

#### Unit testing
- [x] Mocking Data Source from collection with  timestamp assigner
- [x] Mocking Sink to store data and get back when processing completed

#### Integration Testing

- EmbeddedKafka
- Embedded Elasticsearch (Test helpers will be provided)

### CI
**Jenkinsfile will be shared soon**

- Build Job
- Run tests (using embedded kafka and embedded elasticsearch in my case)
- Filter running jobs to get `current job id` using `Flink REST API`
```sh
curl http://localhost:8081 | ./jq '.jobs[] | select(.name == "Awesome Job") | .jid'
```
- Cancel job with savepoint
- Upload new `{Your Job name-version}.jar`
- Run newly uploaded job by starting from previously saved savepoint
