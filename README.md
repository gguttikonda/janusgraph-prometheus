# README #

A simple setup to export JanusGraph JMX metrics as Prometheus endpoint and a utility Grafana dashboard to read metrics.

This setup uses

* [JanusGraph](https://janusgraph.org/) as [Kubernetes](https://kubernetes.io) service deployment.
* [JanusGraph JMX metrics](https://docs.janusgraph.org/advanced-topics/monitoring/#jmx-reporter)
* [JMX Prometheus exporter](https://github.com/prometheus/jmx_exporter).
* [Prometheus Operator](https://github.com/helm/charts/tree/master/stable/prometheus-operator)

## Dependencies ##

* JanusGraph
* Kubernetes
* CoreOS Prometheus


## Setup ##

### ConfigMap ###
Upload the [jmx_exporter](https://github.com/prometheus/jmx_exporter) and [config.yml](https://github.com/gguttikonda/janusgraph-prometheus/blob/master/janusgraph/config.yml) as ConfigMap

```
namespace='database'
kubectl create configmap janusgraph-prom-config --namespace=$namespace
kubectl create configmap janusgraph-prom-config --namespace=$namespace --from-file=janusgraph/  -o yaml --dry-run | kubectl replace -f -
```

### Deployment ###

```
kubectl apply -f janusgraph/janusgraph-deployment.yaml
```

### Service ###
```
kubectl apply -f common/janusgraph-service.yaml
```


### ServiceMonitor ###
```
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  namespace: monitoring
  name: jg-metrics-reader
  labels:
    team: backend
spec:
  jobLabel: team
  selector:
    matchLabels:
      team: backend
  namespaceSelector:
    matchNames:
    - database
  endpoints:
  - port: jg-prom-port
    interval: 15s
    path: /metrics

```

### Grafana ###
* `datasource` in Dashboard [json](https://github.com/gguttikonda/janusgraph-prometheus/blob/master/janusgraph/janusgraph-dashboard.json) is hard-wired `prometheus`.
* ![Screenshot](https://github.com/gguttikonda/janusgraph-prometheus/blob/master/janusgraph/janusgraph-dashboard.png)
