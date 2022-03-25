# On-Prem Monitoring Stack

Out of cluster monitoring via prometheus, fluent-bit and elasticsearch

### Minimum VM Specifications
VM - Prometheus 192.168.101.130 - 2Gb RAM, 2 cores
VM - Grafana 192.168.101.140 - 2Gb RAM, 2 cores
VM - Elasticsearch and kibana 192.168.101.150 - 5Gb RAM, 4 cores

### Recommended VM Specifications for production workloads
VM - Prometheus 192.168.101.130 - 8Gb RAM, 4 cores
VM - Grafana 192.168.101.140 - 4Gb RAM, 4 cores
VM - Elasticsearch and kibana 192.168.101.150 - 16Gb RAM, 4 cores

### Data persistence

Highly dependant on workload, debug settings, especially for log data

Prometheus Data Disk - 128Gb - should be enough for years
Elasticsearch Data Disk - 256Gb - likely 3-6 months

## Installation of Prometheus and Grafana servers

Adjust inventory as necessary, specifically the scraping configuration

```bash
ansible-playbook -i inventory/test metrics.yaml
```

## Installation of exporters on kubernetes clusters

### Helm

customised the values.yaml file for this instance
 - disable alertmanager
 - disable push-gateway and
 - disabled data persistence
 - hanged the port number for the exporter to 9101 from 9100 to not conflict with lighttpd
 - changed the prometheus server service to "LoadBalancer" type, which would be correct for on-prem with metallb

Alternative options would be to enable Ingress, if ambasaddor or similiar is available, but that is up to you

The Prometheus aggregation server is expecting to be able to contact the prometheus running on the cluster, thus
anything that makes the federation endpoint available from the prometheus server would be acceptable

### Installation process

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prom prometheus-community/prometheus --kube-context atlantis -n monitoring --create-namespace -f ./helm-charts/kube-prometheus/values.yaml
```

## Installation of metrics-server on the cluster (might not be necessary, but it does enrich kubectl)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml
```

## ElasticSearch and Kibana

### Installation of Elasticsearch and Kibana

Customise the inventory as required

```bash
ansible-playbook -i inventory/test logs.yaml
```

## Kubenetes cluster log aggregation

### For each cluster

### Fluent-bit

https://github.com/fluent/helm-charts/tree/main/charts/fluentd

### Fluentbit is specifically built for kubernetes, works out of the box

Update the values.yaml for fluent-bit chart with the elastic search server information

helm install fluent-bit ./helm-charts/fluent/fluent-bit --kube-context atlantis --namespace fluent --create-namespace -f values.yaml

## MongoDB Exporter

The metrics exporter is installed by the playbook when deploying mongodb

Manual testing

```
sudo /usr/local/bin/mongodb_exporter --web.listen-address=[0.0.0.0]:9217 --collect.database --collect.topmetrics --collect.indexusage --collect.connpoolstats --mongodb.uri=mongodb://metrics-exporter:XXXXXXXXXXX@192.168.101.121
```

## RabbitMQ Exporter

The metrics exporter is installed by the playbook when deploying rabbitmq