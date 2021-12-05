## Mutating  input

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
<td> Mutating dummy input
<td> 

```
# Mutating event filter
# Add hostname and tag fields to dummy tag events
<filter dummy>
	@type record_transformer
	<record>
		hostname ${hostname}
		tag ${tag}
	</record>
</filter>
```

```
kubectl create configmap fluentd-config --from-file fluentd-kube.conf
```

Fluentd logs:
``` bash
2021-11-10 19:35:05.008798500 +0000 dummy: {"hello":"fluentd","hostname":"fluentd-ds-pvc-bzskk","tag":"dummy"}
2021-11-10 19:35:06.010864500 +0000 dummy: {"hello":"fluentd","hostname":"fluentd-ds-pvc-bzskk","tag":"dummy"}
2021-11-10 19:35:07.012942600 +0000 dummy: {"hello":"fluentd","hostname":"fluentd-ds-pvc-bzskk","tag":"dummy"}
```


</table>