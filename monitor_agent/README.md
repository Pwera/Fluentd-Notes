## monitor agent

<table>
<tr>
<td> Install fluentd as a daemonset
<td> 

``` yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-ds
  labels:
    k8s-app: fluentd-logging
    version: v1
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-logging
  template:
    metadata:
      labels:
        k8s-app: fluentd-logging
        version: v1
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      terminationGracePeriodSeconds: 30
      containers:
      - name: fluentd-ds
        image: fluent/fluentd:latest
        resources:
          limits:
            memory: 200Mi
        env:
          - name:  FLUENTD_CONF
            value: "fluentd-kube.conf"
        volumeMounts:
        - name: fluentd-conf
          mountPath: /fluentd/etc
      volumes:
      - name: fluentd-conf
        configMap:
          name: fluentd-config
```

```
kubectl apply -f  fluentd-daemonset.yaml
```

<tr>
<td> Add monitor_agent
<td> 

```
# Listen HTTP for monitoring
# http://localhost:24222/api/plugins
# http://localhost:24222/api/plugins?type=TYPE
# http://localhost:24222/api/plugins?tag=MYTAG
<source>
	@type monitor_agent
	@id monitor_agent_input
	port 24222
</source>
```


``` bash
kubectl create configmap fluentd-config --from-file fluentd-kube.conf
```
Verify output:

```bash
curl http://<POD_IP>:24222/api/plugins

plugin_id:object:2b2026769f78   plugin_category:input   type:http       output_plugin:false     retry_count:
plugin_id:object:2b20261801ac   plugin_category:input   type:forward    output_plugin:false     retry_count:
plugin_id:object:2b20267c1e44   plugin_category:input   type:tail       output_plugin:false     retry_count:
plugin_id:object:2b20267851ec   plugin_category:input   type:dummy      output_plugin:false     retry_count:
plugin_id:monitor_agent_input   plugin_category:input   type:monitor_agent      output_plugin:false     retry_count:
plugin_id:object:2b20266bc01c   plugin_category:output  type:stdout     output_plugin:true      retry_count:0
plugin_id:object:2b20266b47b8   plugin_category:filter  type:record_transformer output_plugin:false     retry_count:
```

</table>
