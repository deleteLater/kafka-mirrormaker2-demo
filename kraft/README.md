## Setup

### Strimzi

```shell
kubectl version --output=json

kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
kubectl get pod -n kafka --watch
kubectl logs deployment/strimzi-cluster-operator -n kafka -f
```

### Enabling KRaft

```bash
kubectl -n kafka set env deployment strimzi-cluster-operator STRIMZI_FEATURE_GATES=+KafkaNodePools,+UseKRaft
kubectl -n kafka get deployment strimzi-cluster-operator -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="STRIMZI_FEATURE_GATES")]}' | jq
```

KRaft and its various architectures were one of the main motivations when designing node pools.

The Kafka nodes in the KRaft mode have controller and/or broker roles. The nodes with the controller role are
responsible for maintaining the Kafka metadata, handling leader elections, and helping to bootstrap the Kafka cluster.
They assume the responsibilities previously handled by the ZooKeeper nodes. The nodes with the broker role have the same
responsibilities as the Kafka brokers in the ZooKeeper-based Kafka cluster. They receive the messages from the
producers, store them, and pass them to the consumers. One node can also have both roles - a controller and a broker -
at the same time. This offers a number of configuration options for your KRaft-based Kafka cluster.

## References

- [Kafka Node Pools: Supporting KRaft](https://strimzi.io/blog/2023/09/11/kafka-node-pools-supporting-kraft/)
- [Kafka Node Pools Examples](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka/nodepools)
- [Node Pools Demo: With KRaft](https://www.youtube.com/watch?v=_yx-0IWNsHc)