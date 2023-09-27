# Kafka MirrorMaker 2.0 Local Demo

## Setup Kafka Clusters

### Clone

```shell
git clone https://github.com/deleteLater/kafka-mirrormaker2-demo.git
cd kafka-mirrormaker2-demo
```

### Strimzi

```shell
kubectl version --output=json

kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
kubectl get pod -n kafka --watch
kubectl logs deployment/strimzi-cluster-operator -n kafka -f
```

### Kafka clusters

```shell
kubectl apply -f kafka-source.yaml -n kafka
kubectl wait kafka/cluster-a --for=condition=Ready --timeout=300s -n kafka

kubectl apply -f kafka-target.yaml -n kafka
kubectl wait kafka/cluster-b --for=condition=Ready --timeout=300s -n kafka
```

> **Note**
> If failed to start zookeeper while starting kafka cluster, delete the pod and try again.

### Kafka ui

```shell
kubectl create -f kafka-ui.yaml -n kafka
kubectl get pod -n kafka --watch

kubectl port-forward service/kafka-ui-service 8080:8080 -n kafka --address='0.0.0.0'
```

## Local MirrorMaker

### Setup

```shell
kubectl create -f local-mirror-maker-2.yaml -n kafka
kubectl get pod -n kafka -watch
```

### Test

```shell
kubectl -n kafka exec --stdin --tty cluster-a-kafka-0 -- /bin/bash
kubectl -n kafka exec --stdin --tty cluster-b-kafka-0 -- /bin/bash

# list kafka topics
./bin/kafka-topics.sh --list --bootstrap-server cluster-a-kafka-bootstrap:9092
./bin/kafka-topics.sh --list --bootstrap-server cluster-b-kafka-bootstrap:9092

# produce & consume message from cluster-a
bin/kafka-console-producer.sh --bootstrap-server cluster-a-kafka-bootstrap:9092 --topic my-topic
bin/kafka-console-consumer.sh --bootstrap-server cluster-a-kafka-bootstrap:9092 --topic my-topic --from-beginning

# consumer message from cluster-b
bin/kafka-console-consumer.sh --bootstrap-server cluster-b-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

## Remote MirrorMaker

### Setup

```shell
# on laptop (address is 192.168.1.163), expose cluster-b-kafka
kubectl port-forward service/cluster-b-kafka-bootstrap 9092:9092 -n kafka --address='0.0.0.0'

# on pc, list remote cluster-b-kafka's topics
kubectl -n kafka exec --stdin --tty cluster-a-kafka-0 -- /bin/bash
./bin/kafka-topics.sh --list --bootstrap-server 192.168.1.163:9092

# on pc, create remote mirror maker
kubectl create -f remote-mirror-maker-2.yaml -n kafka
kubectl get pod -n kafka -watch

kubectl -n kafka exec --stdin --tty cluster-a-kafka-0 -- /bin/bash

# produce & consume message from pc's cluster-a
bin/kafka-console-producer.sh --bootstrap-server cluster-a-kafka-bootstrap:9092 --topic my-topic
bin/kafka-console-consumer.sh --bootstrap-server cluster-a-kafka-bootstrap:9092 --topic my-topic --from-beginning

# consumer message from laptop's cluster-b
kubectl -n kafka exec --stdin --tty cluster-b-kafka-0 -- /bin/bash
bin/kafka-console-consumer.sh --bootstrap-server cluster-b-kafka-bootstrap:9092 --topic my-topic --from-beginning
```

### Test

TODO