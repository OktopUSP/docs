---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# High Availability

For deployments with failover and advanced container management/orchestration, Kubernetes and it's flavors are the go-to option. This section aims to provide a example deployment that may need to be changed according to your environment specifics, as hardware/software Load Balancers, cluster nodes,  firewall, bare-metal or cloud environment.

## Installation

### Download Files

```
git clone https://github.com/OktopUSP/oktopus
export DEPLOYMENT_PATH=oktopus/deploy/kubernetes
```

### HAProxy Ingress Controller

```
helm install haproxy-kubernetes-ingress haproxytech/kubernetes-ingress \
  --create-namespace \
  --namespace haproxy-controller \
  --set controller.kind=DaemonSet \
  --set controller.daemonset.useHostPort=true
```

### MongoBD

```
# Mongo DB Operator at mongodb namespace
helm repo add mongodb https://mongodb.github.io/helm-charts

helm install community-operator mongodb/community-operator --namespace mongodb --create-namespace

# Mongo DB ReplicaSet
export DEPLOYMENT_PATH=oktopus/deploy/kubernetes

kubectl apply -f $DEPLOYMENT_PATH/mongodb.yaml -n mongodb

# Check Installation
kubectl get pods -n mongodb
```

### NATS Server

```
# Download the NATS charts
helm repo add nats https://nats-io.github.io/k8s/helm/charts/

# Install NATS with Jetstream Enabled
helm install nats nats/nats --set config.jetstream.enabled=true
```

### Oktopus

```
kubectl apply -f $DEPLOYMENT_PATH/mqtt.yaml
kubectl apply -f $DEPLOYMENT_PATH/mqtt-adapter.yaml
kubectl apply -f $DEPLOYMENT_PATH/adapter.yaml
kubectl apply -f $DEPLOYMENT_PATH/controller.yaml
kubectl apply -f $DEPLOYMENT_PATH/socketio.yaml
kubectl apply -f $DEPLOYMENT_PATH/frontend.yaml
kubectl apply -f $DEPLOYMENT_PATH/ws.yaml
kubectl apply -f $DEPLOYMENT_PATH/ws-adapter.yaml
```

### Checking cluster status:

```bash
kubectl get pods
kubectl get svc
```
