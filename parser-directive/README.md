## Parsers

Fluentd can be configured to ingest data from a wide variety of sources; in many cases, those sources will emit logged events in a format that may require some effort to parse and understand (logs often consist of single lines of raw data without headers or delimiters). When Fluentd receives log data, by default, it captures the data as is.

Data parsers allow Fluentd to reorganize unstructured data into a format that can be easily read by humans and machines. Parsers do this by separating the data into unique key-value pairs and representing it in JSON format. The keys can then be used by logic in filter directives and other components of a Fluentd logging pipeline.




<table>

<tr>
<td>Fluentd Parsed Event
<td>


```
cat /var/log/apache2/access.log

127.0.0.1 - - [05/May/2021:15:08:07 +0000] "GET / HTTP/1.1" 200 11173 "-" "curl/7.68.0"
```



```
# Fluentd Parsed Event (Using Apache parser):

20210505T150807+0000,apache2.access,{"host":"127.0.0.1","user":null,"method":"GET","path":"/","code":200,"size":11173,"referer":null,"agent":"curl/7.68.0"}
```


<tr>
<td>No Parsing
<td>

```
<source>
  @type tail
  path /var/log/apache2/access.log
  pos_file /tmp/apache_access.pos
  tag apache2.access
  <parse>
    @type none
  </parse>
</source>
<match>
  @type stdout
</match>

```

```
2021-05-05 15:14:15.719568926 +0000 apache2.access: {"message\"-\"":"127.0.0.1 - - [05/May/2021:15:14:15 +0000] \"GET / HTTP/1.1\" 200 11173 \"curl/7.68.0\""}
```

<tr>
<td>Fluentd Parsing
<td>


```
<source>
  @type tail
  path /var/log/apache2/access.log
  pos_file /tmp/apache_access.pos
  tag apache2.access
  <parse>
    @type apache2
  </parse>
</source>
<match>
  @type stdout
</match>
```

```
2021-05-05 15:31:31.000000000 +0000 apache2.access: {"host":"127.0.0.1","user":null,"method":"GET","path":"/","code":200,"size":11173,"referer":null,"agent":"curl/7.68.0"}
```


</table>