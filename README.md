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

uninstallation

```
helm uninstall prom --kube-context atlantis -n monitoring
```

federated scraping config

scrape_configs:
  - job_name: 'test-cluster'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job="kubernetes-pods"}'
        - '{__name__=~"job:.*"}'

    static_configs:
      - targets:
        - '192.168.101.201:9090'