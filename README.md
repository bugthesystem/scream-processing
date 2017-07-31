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
- [Scala 2.11.7](https://www.scala-lang.org/)
- [Sbt 0.13](http://www.scala-sbt.org/)
- [Kafka 0.10](https://kafka.apache.org/)
- [Flink 1.3.1](https://flink.apache.org/)
  - [FlinkCEP - Complex event processing for Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.2/dev/libs/cep.html)
  - [FlinkML - FlinkML - Machine Learning for Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.2/dev/libs/ml/index.html)
- [Elasticsearch 5.5.0](https://www.elastic.co/products/elasticsearch)
- [Kibana 5.5.0](https://www.elastic.co/products/kibana)
- [Docker](https://www.docker.com/)
- [Kubernetes 1.7](https://kubernetes.io/)
- [Jenkins Pipelines](https://jenkins.io/doc/book/pipeline/)

# Env Setup (Local)
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
```

## Install Flink
```sh
wget http://mirror.netinch.com/pub/apache/flink/flink-1.3.0/flink-1.3.1-bin-hadoop27-scala_2.11.tgz

tar xzf flink-*.tgz   # Unpack the downloaded archive
cd flink-1.3.1

# Start Flink
./bin/start-local.sh

#OR
docker pull flink

docker run -t -p 8081:8081 flink local
```

## Install ES/Kibana
:warning: [Flink 1.3.0] `Flink Elasticsearch connector` for `Elasticsearch 5` is missing in `Maven` repository atm.

```sh
#Elasticsearch 5.5.0
docker run --name scream-processing-elasticsearch -p 9200:9200 -p 9300:9300 \
           -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" -d elasticsearch:5.5.0

# Kibana 5.5.0
docker run --name scream-processing-kibana --link scream-processing-elasticsearch:elasticsearch -p 5601:5601 -d kibana:5.5.0
```

# Env Setup (Kubernetes)

## Flink
```sh
# clone docker-flink/examples repo
git clone git@github.com:docker-flink/examples.git
 
cd docker-flink
 
# Build the Helm archive:
helm package helm/flink/

# Create namespace for `flink`
kubectl create ns flink

# Deploy a non-HA Flink cluster with a single taskmanager:
helm install --name scream-processing-flink  --set global.namespace=flink flink*.tgz
 
# Deploy a non-HA Flink cluster with three taskmanagers:
helm install --name scream-processing-flink --set flink.num_taskmanagers=3 --set global.namespace=flink flink*.tgz
 
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
helm install --name scream-processing-flink --set global.namespace=flink --values values.yaml flink*.tgz
```

## Kafka
```sh
# Add helm repo
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
 
# Create namespace for `kafka`
kubectl create ns kafka
 
# Installs
helm install --name scream-processing-kafka --set global.namespace=kafka incubator/kafka
 
# Delete
helm delete scream-processing-kafka
```

# Project Setup

## From stratch /w sbt
```sh
# Run following script ot clone starter repo
bash <(curl https://flink.apache.org/q/sbt-quickstart.sh)
```
## Using boilerplate
```sh
# TODO:
```

## Project structure and dependencies
```sh
# TODO: boilerplate project will be shared
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

- [x] Build Job
- [x] Run tests (using embedded kafka and embedded elasticsearch in my case)
- [x] Filter running jobs to get `current job id` using `Flink REST API`
```sh
curl http://localhost:8081 | ./jq '.jobs[] | select(.name | startswith("Awesome Job")) | .jid'
```
- [x] Cancel job with savepoint
- [x] Upload new `{Your Job name-version}.jar`
- [x] Run newly uploaded job by starting from previously saved savepoint

### Warnings
 - If you want to upload fat-jars and if you get `413` (Entity Too Large) add following annotations to your ingress.
 
 ```yaml
kind: Ingress
metadata:
  annotations:
    ingress.kubernetes.io/proxy-body-size: <your max size>m
    nginx.org/client-max-body-size: <your max size>m
 ```
