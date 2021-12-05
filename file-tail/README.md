## File tail


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
<td> Add http input
<td> 

```
<source>  
	@type http  
	port 24220  
	bind 0.0.0.0
</source>
```


<tr>
<td> Add configuration as a configmap
<td>

``` bash
kubectl create configmap fluentd-config --from-file fluentd-kube.conf
```


<tr>
<td> Check logs 
<td> 

``` bash
kubectl logs $(kubectl get pod -l=k8s-app=fluentd-logging -o custom-columns=:metadata.name) -f
```