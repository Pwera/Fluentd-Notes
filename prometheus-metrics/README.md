## Configuring Prometheus Monitoring with Fluentd


```
https://raw.githubusercontent.com/fluent/fluent-plugin-prometheus/master/misc/prometheus.yaml

# A job to scrape an endpoint of Fluentd running on localhost.
scrape_configs:
- job_name: 'prometheus'
scrape_interval: 5s
static_configs:
- targets:
- 'prometheus:9090'
- job_name: fluentd
scrape_interval: 5s
static_configs:
- targets:
- 'fluentd-ds-pvc-svc:24231'
metrics_path: /metrics

```

```
<source>
  @type prometheus
</source>
<source>
  @type prometheus_monitor
</source>
<source>
  @type forward
</source>
<filter mod8.*>
  @type prometheus
  <metric>
    name fluentd_retrieved_status_codes_count
    type histogram
    desc The total number of status code-bearing requests
    key message
  </metric>
</filter>
<match>
  @type file
  path /tmp/fluentd.monitored.log
</match>

```

```
kubectl create configmap fluentd-config --from-file fluentd-kube.conf
kubectl apply -f .
```

```
echo '{"message":'$RANDOM'}' | fluent-cat mod8.prometheus
```

```
curl -s localhost:24231/metrics | grep fluentd_retrieved_status_codes
```

```
# TYPE fluentd_retrieved_status_codes histogram
# HELP fluentd_retrieved_status_codes The total number of status code-bearing requests
...
fluentd_retrieved_status_codes_sum 13465.0
fluentd_retrieved_status_codes_count 1.0
```

