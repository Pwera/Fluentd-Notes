

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


<filter my.tag>
  @type grep
  regexp1 message example
</filter>

<filter my.tag>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<filter some.tag>
  @type record_transformer
  enable_ruby
  <record>
    avg ${record["total"] / record["count"]}
  </record>
</filter>


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
echo '{"message":"hello"}'| fluent-cat mod2.lab -p 31604 --json
```

```
0000 mod2.lab: {"message":"hello","event_tag":"mod2.lab"}

```


```
curl -X POST -d 'json={"message":"hello from http"}' http://localhost:32767/mod2.http
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

``` conf
<source>
  @type forward
</source>
 <filter **>
    @type grep
    <regexp>
      key "event"
      pattern /actual/
    </regexp>
  </filter>
<match>
  @type stdout
</match>
```

``` bash
echo '{"event":"actual","data":"valid"}' | fluent-cat record ; echo '{"event":"test","data":"valid"}' | fluent-cat record
```

``` conf
2021-05-04 16:45:33.842902363 -0700 record: {"event":"actual","data":"valid"}
```

<tr>
<td>
<td>

```
<filter my.tag>
  @type record_transformer
  <record>
    msg ${record["msg"]} world
    hostname "#{Socket.gethostname}"
  </record>
</filter>
```

```
{"msg":"hello"}
{"msg":"hello world", "hostname":"host.example.com"}

```

```
```

<tr>
<td>

```
filter_record_transformer
auto_typecast
enabled
```
<td>

```
<filter mod4**>
  @type record_transformer
  enable_ruby true
  <record>
    avg ${record["total"] / record["count"]}
  </record>
</filter>
```

```
echo '{"total":50,"count":50}' | fluent-cat mod4
```

```
{"total":50,"count":50,"avg":1}
```

<tr>
<td>

```
filter_record_transformer
auto_typecast
disabled
```

<td>

```
<filter mod4**>
  @type record_transformer
  enable_ruby true
  <record>
    avg ${record["total"] / record["count"]}
  </record>
  auto_typecast false
</filter>
```

```
echo '{"total":50,"count":50}' | fluent-cat mod4
```

```
{"total":50,"count":50,"avg":"1"}
```


<tr>
<td>

```
filter_record_transformer
renew_record
```

<td>
filter_record_transformer provides the renew_record parameter which, when set to true, creates a new empty record that replaces the original record. This is useful when none or few of the original record fields need to be forwarded.

The keep_keys parameter can be supplied with the names of keys (separated by commas) to transfer from the original record to the new record. The <record> subdirective can also be used as usual to create new fields in the new record. The record[] array can be used to access values from the original record when composing fields in the new record.


```
<filter mod4**>
  @type record_transformer
  enable_ruby
  <record>
    action ${record["log"]}
  </record>
  renew_record true
  keep_keys weather
</filter>
```


```
echo '{"time":"today","weather":"rain","log":"stayed inside"}' | fluent-cat mod4
```

```
{"weather":"rain","action":"stayed inside"}
```

<tr>
<td> 

```
filter_record_transformer
renew_time_key
```
<td>

The filter_record_transformer renew_time_key parameter overwrites the message timestamp with the value of a named record field. The time value must be in Unix time format.

```
<filter mod4**>
  @type record_transformer
  enable_ruby
  <record>
    reality ${record["event"]}
  </record>
  renew_time_key timetag
</filter>
```

```
echo '{"event":"real"}' | fluent-cat mod4
```

```
2021-05-04 16:49:18.677062634 -0700 mod4: {"event":"real","reality":"real"}
```


<tr>
<td>

```
filter_record_transformer
remove_keys
```

<td>

```
<filter **>
  @type record_transformer
  enable_ruby
  <record>
    message login - ${record["username"]}
  </record>
  remove_keys username
</filter>
```

```
echo '{"username":"student"}' | fluent-cat event
```


```
2021-05-04 16:51:18.257313920 -0800 event: {"message":"login - student"}
```


<tr>
<td>

```
filter_grep
```

<td>

The grep filter directive is configured using subdirectives:

```<regexp> ```- forwards events that match the regexp pattern.
```<exclude>``` - drops events that match the regexp pattern.

Both ```<regexp>``` and ```<exclude>``` use the "key" and "pattern" parameters. The ```<regexp>``` subdirective defines a field (key) and a regular expression (pattern) to match. Records having a value for the given key matching the given pattern are forwarded; other records are dropped. The ```<exclude>``` subdirective performs the opposite task, dropping records that do match.

```
<filter my.tag>
  @type grep
  <regexp>
    key message
    pattern /string-literal/
  </regexp>
  <regexp>
    key hostname
    pattern /^db\d+\.example\.com$/
  </regexp>
  <exclude>
    key message
    pattern /other-string/
  </exclude>
</filter>
```

By default, all subdirectives must be satisfied (logical AND) to forward the record. The ```<and>``` and ```<or>``` subdirectives can be used to change this logic:

```<and>``` - All regexp and exclude patterns must be satisfied.
```<or>``` - Only one of the enclosed patterns needs be satisfied.


<tr>
<td>

```
filter_parser
```
<td>
The filter_parser plugin parses a given key’s string value, replacing the top level fields in the record with the parsed fields from the string:

```key_name``` – the name of the record to parse.
```<parse>``` – the parser to use (JSON, CSV, msgpack, TSV, nginx, apache2, etc.). 
Original record data can be preserved with reserve_* parameters:

```reserve_time``` – keeps the original timestamp (otherwise current time is used).
```reserve_data``` – keeps the original record data (appending parsed key-value data).
```remove_key_name_field``` – removes the parsed key-value data from the record.


```
<filter my.tag>
  @type parser
  key_name user
  reserve_data true
  remove_key_name_field true
  <parse>
    @type json
  </parse>
</filter>
```

```
{"key":"value","user":"{\"foo\":1,\"bar\":2}"}
```

```
{"key":"value","foo":1,"bar":2}
```


<tr>
<td>

```
filter_stdout
```
<td>

The filter_stdout plugin prints matching events to stdout. This plugin is helpful for debugging, as it can be used to check post-filter output at any point in the data processing flow.

The filter_stdout plugin can use the ```<format>``` subdirective to call a formatter plugin to format the outgoing event. The filter_stdout plugin also supports the ```<inject>``` subdirective for injecting values into the event record

</table>