# Kafka MirrorMaker 2.0 Local Demo

## Setup

```shell
minikube version
kubectl version --output=json

minikube start --memory=8g --cpus=4
kubectl get nodes -o wide

kubectl create namespace kafka
kubectl create -f 'https://strimzi.io/install/latest?namespace=kafka' -n kafka
kubectl get pod -n kafka --watch
kubectl logs deployment/strimzi-cluster-operator -n kafka -f

kubectl apply -f kafka-source.yaml -n kafka
kubectl apply -f kafka-target.yaml -n kafka
kubectl get pod -n kafka --watch

kubectl create -f kafka-ui.yaml -n kafka
kubectl get pod -n kafka --watch

kubectl port-forward service/kafka-ui-service 8080:8080 -n kafka

kubectl create -f local-mirror-maker-2.yaml -n kafka
kubectl get pod -n kafka -watch
```
