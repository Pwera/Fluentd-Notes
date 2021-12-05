

The <match> directive configures an output plugin that will send data to an intended destination, which can be an external datastore, a
file, or back into Fluentd for further processing. The @type parameter within a <match> directive will select plugins with the out_
prefix.
The formatting of the <match> directive is different in that the opening tag can include a pattern. This pattern determines what event
tags will be processed by that <match> directive. If no pattern is specified, all events will be processed by that <match> directive.


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
<td> Configuration
<td>

``` conf
<source>  
	@type http  
	port 24220  
	bind 0.0.0.0

</source>
<source>  
	@type forward  
	port 24224  
	bind 0.0.0.0
</source>

<source>
	@type tail
	path /tmp/fluent/*.log
	<parse>
		@type none
	</parse>
	tag local.logs
	pos_file /tmp/fluent/lab2.tail.pos
</source>

<match **>
	@type stdout
	<inject>
		tag_key event_tag
	</inject>
</match>


<match **>  
	@type stdout
</match>

```



``` bash
kubectl create configmap fluentd-config --from-file fluentd-kube.conf
```


<tr>
<td> Test configuration
<td>

```
echo '{"message":"hello"}'| fluent-cat mod2.lab -p 24224 --json
```

```
0000 mod2.lab: {"message":"hello","event_tag":"mod2.lab"}

```


```
curl -X POST -d 'json={"message":"hello from http"}' http://localhost:24220/mod2.http
```

```
+0000 mod2.http: {"message":"hello from http","event_tag":"mod2.http"}
```

```
echo 'An error has occurred: Done' >> /tmp/fluent/error.log
```

```
+0000 [warn]: #0 no patterns matched tag="local.logs"
```

<tr>
<td>
<td>

<tr>
<td>
<td>


</table>