# On-Prem Monitoring Stack

In cluster kube monitoring via prometheus + server

VM with prometheus 192.168.101.130
VM with grafana 192.168.101.120

## Installs prometheus and grafana on VM's

`ansible-playbook -i inventory/test site.yaml -u install`

## Install prometheus on kubecluster to grab pod metrics

helm based

testing the below, to be automated

customised the values.yaml file to disable alertmanager and push-gateway and data persistence
also changed the port number for the exporter to 9101 from 9100 to not conflict with httpd

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prom prometheus-community/prometheus --kube-context atlantis -n monitoring --create-namespace -f kube-prometheus/values.yaml
```

## Uninstall the K8s prometheus

```
helm uninstall prom --kube-context atlantis -n monitoring
```

## Install metrics-server on the cluster

kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml

## Fluentd

https://github.com/fluent/helm-charts/tree/main/charts/fluentd


helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
helm install fluentd fluent/fluentd


## ElasticSearch

### Setup: Elasticsearch and Kibana

playbook works for deployment to VMs

### Setup fluentd aggregation

helm install fluent . --kube-context atlantis --namespace fluent --create-namespace -f values.yaml


curl -L http://toolbelt.treasuredata.com/sh/install-ubuntu-precise-td-agent2.sh | sh

$ sudo /usr/sbin/td-agent-gem install fluent-plugin-secure-forward
$ sudo /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch

td-agent.conf

# Listen to incoming data over SSL
<source>
  type secure_forward
  shared_key FLUENTD_SECRET
  self_hostname logs.example.com
  cert_auto_generate yes
</source>

# Store Data in Elasticsearch and S3
<match *.**>
  type copy
  <store>
    type elasticsearch
    host localhost
    port 9200
    include_tag_key true
    tag_key @log_name
    logstash_format true
    flush_interval 10s
  </store>
  <store>
    type s3
    aws_key_id AWS_KEY
    aws_sec_key AWS_SECRET
    s3_bucket S3_BUCKET
    s3_endpoint s3-ap-northeast-1.amazonaws.com
    path logs/
    buffer_path /var/log/td-agent/buffer/s3
    time_slice_format %Y-%m-%d/%H
    time_slice_wait 10m
  </store>
</match>

sudo service td-agent restart