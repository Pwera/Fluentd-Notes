## Pass TCP and HTTP traffic to stdout

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

<tr>
<td> Test configuration
<td> 

``` bash
curl -X POST -d 'json={"Hello":"FluentD"}' http://<POD_IP>:24220/test

Fluentd logs:
2021-11-10 19:24:15.382412500 +0000 fluent.info: {"worker":0,"message":"fluentd worker is now running worker=0"}
2021-11-10 19:25:43.003844400 +0000 test: {"Hello":"FluentD"}
```

<tr>
<td>Tcp input
<td>

```
<source>
	@type forward
	port 24224
	tag forward
</source>
```

```
echo {"message":"hello"} | fluent-cat forward -p 24224 --none
```

Fluentd logs:

``` bash
2021-11-10 20:43:37.554230200 +0000 forward: {"message":"{message:hello}"}
```
<tr>
<td>
Test commands:
<td>

```
kubectl run test-client --image centos -it -- curl -X POST -d 'json={"Hello":"World"}' http://172.17.0.3:24220/test
```

</table>