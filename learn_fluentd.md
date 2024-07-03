# Learn Fluentd

## Install

```
gem install fluentd

gem install fluentd-plugin-grafana-loki
```


## Send Logs to Loki

vi fluent.conf
```
<source>
  @type tail
  format none
  path /var/log/*.log
  pos_file positions.pos
  tag varlog.*
  path_key filename
</source>

<match vatlog.**>
  @type loki
  url "http://localhost:3100"
  flush_interval 1s
  flush_at_shutdown true
  buffer_chunk_limit 1m
  extra_labels {"job":"varlogs", "host":"ward_workstation", "agent":"fluentd"}
  <label>
    filename
  </label>
</match>

```

