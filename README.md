# On-Prem Monitoring Stack

Out of cluster monitoring via prometheus, fluent-bit and elasticsearch

VM with prometheus 192.168.101.130 - 2Gb RAM, 2 core
VM with grafana 192.168.101.140 - 2Gb RAM, 2 core
VM with elasticsearch and kibana 192.168.101.150 - 5Gb RAM, 4 core

## Installs prometheus and grafana on VM's

`ansible-playbook -i inventory/test metrics.yaml -u install`

## Install prometheus on the kubernetes cluster to grab pod metrics

helm based

testing the below, to be automated

customised the values.yaml file for this instance
 - disable alertmanager
 - disable push-gateway and
 - disabled data persistence
 - hanged the port number for the exporter to 9101 from 9100 to not conflict with lighttpd
 - changed the prometheus server service to "LoadBalancer" type, which would be correct for on-prem with metallb

Alternative options would be to enable Ingress, if ambasaddor or similiar is available, but that is up to you

The Prometheus aggregation server is expecting to be able to contact the prometheus running on the cluster, thus
anything that makes the federation endpoint available from the prometheus server would be acceptable

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prom prometheus-community/prometheus --kube-context atlantis -n monitoring --create-namespace -f ./helm-charts/kube-prometheus/values.yaml
```

## Install metrics-server on the cluster (might not be necessary, but it does enrich kubectl)

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

## ElasticSearch and Kibana

### Setup: Elasticsearch

Customise the inventory as required

`ansible-playbook -i inventory/test logs.yaml -u install`

### Setup : Kibana

Customise the inventory as required

`ansible-playbook -i inventory/test logs.yaml -u install`


## Setup kubenetes cluster log aggregation

### For each cluster

## Fluent-bit

https://github.com/fluent/helm-charts/tree/main/charts/fluentd

### Fluentbit is specifically built for kubernetes, works out of the box

Update the values.yaml for fluent-bit chart with the elastic search server information

helm install fluent-bit ./helm-charts/fluent/fluent-bit --kube-context atlantis --namespace fluent --create-namespace -f values.yaml