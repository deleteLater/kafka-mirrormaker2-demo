# Kafka MirrorMaker 2.0 Local Demo

## Notes

I have a PC and a laptop in a local area network, both installed with docker and Kubernetes.

- PC's ip    : 192.168.1.93
- Laptop's ip: 192.168.1.163

These ip address will be used in the following files

- kafka-target.yaml
- remote-mirror-maker-2.yaml

And I run the `test1` folder's yaml files on my pc, and `test2` folder's yaml files on my laptop.

## Setup Kafka Clusters

### Clone

```shell
git clone https://github.com/featbit/kafka-mirrormaker2-demo.git
cd kafka-mirrormaker2-demo
cd test1 # or test2
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
> Sometimes we may fail to start zookeeper because of the weird coredns problem, in such case, delete the pod and try again.

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
kubectl logs pod/local-mirror-maker-2-mirrormaker2-0 -n kafka -f
```

### Test

We can test the local mirror maker in a terminal

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

Or we can test in Kafka UI

1. create a topic in cluster-a, and the topic should be synced to cluster-b in a short time
2. produce a message in cluster-a, and the message should be synced to cluster-b

## Remote MirrorMaker

### Setup

On PC and laptop, we need to expose these kafka service to the local network

```shell
kubectl port-forward service/cluster-b-kafka-external-bootstrap 9094:9094 -n kafka --address='0.0.0.0'
kubectl port-forward service/cluster-b-kafka-brokers 9092:9092 -n kafka --address='0.0.0.0'
kubectl port-forward service/kafka-ui-service 8080:8080 -n kafka --address='0.0.0.0'
```

After we expose these kafka services

```shell
kubectl create -f remote-mirror-maker-2.yaml -n kafka
kubectl get pod -n kafka -watch
kubectl logs pod/remote-mirror-maker-2-mirrormaker2-0 -n kafka -f
```

### Test

Using the Kafka UI, we can

1. create a topic in cluster-a and the topic should be synced to remote-cluster-b in a short time
2. produce a message in cluster-a and the message should be synced to remote-cluster-b

## References

- [**Kafka MirrorMaker 2 cluster configuration**](https://access.redhat.com/documentation/en-us/red_hat_amq_streams/2.4/html/configuring_amq_streams_on_openshift/assembly-deployment-configuration-str#assembly-mirrormaker-str)
- [**Kafka Connect cluster configuration**](https://access.redhat.com/documentation/en-us/red_hat_amq_streams/2.4/html/configuring_amq_streams_on_openshift/assembly-deployment-configuration-str#assembly-kafka-connect-str)
- [**Strimzi Examples**](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/mirror-maker)
- https://www.uber.com/en-JP/blog/kafka/
