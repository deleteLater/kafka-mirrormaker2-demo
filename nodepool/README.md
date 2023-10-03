## Setup

### Strimzi

```shell
kubectl version --output=json

kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
kubectl get pod -n kafka --watch
kubectl logs deployment/strimzi-cluster-operator -n kafka -f
```

### Enabling node pools

```bash
kubectl -n kafka set env deployment strimzi-cluster-operator STRIMZI_FEATURE_GATES=+KafkaNodePools
kubectl -n kafka get deployment strimzi-cluster-operator -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="STRIMZI_FEATURE_GATES")]}' | jq
```

### Test

The [kafka-using-nodepool.yaml](kafka-using-nodepool.yaml) example shows a regular Kafka cluster backed by ZooKeeper. This example file deploys a
ZooKeeper cluster with 3 nodes and 2 different pools of Kafka brokers. Each of the pools has 3 brokers. The pools in the
example use different storage configuration.

Kafka pods are not created based on the Kafka resource, but based on the KafkaNodePool resources.

## References

- [Kafka Node Pools Proposal](https://github.com/strimzi/proposals/blob/main/050-Kafka-Node-Pools.md#kafka-node-pools)
- [Kafka Node Pools Introduction](https://strimzi.io/blog/2023/08/14/kafka-node-pools-introduction/)
- [Kafka Node Pools Examples](https://github.com/strimzi/strimzi-kafka-operator/tree/main/examples/kafka/nodepools)
- [Node Pools Demo: With ZooKeeper](https://www.youtube.com/watch?v=KznxXsQNlq8)