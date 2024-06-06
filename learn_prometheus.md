# Prometheus

## Install

```
groupadd --system prometheus
useradd -s /sbin/nologin --system -g prometheus prometheus
mkdir /var/lib/prometheus
for i in rules rules.d files_sd; do     sudo mkdir -p /etc/prometheus/${i}; done
cd /etc/prometheus/
cd /opt
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0-rc.1/prometheus-2.52.0-rc.1.linux-amd64.tar.gz
dnf install tar
tar xvzf prometheus-2.52.0-rc.1.linux-amd64.tar.gz
rm prometheus-2.52.0-rc.1.linux-amd64.tar.gz
cd prometheus-2.52.0-rc.1.linux-amd64/
cp prometheus promtool /usr/local/bin/
cp -r prometheus.yml consoles/ console_libraries/ /etc/prometheus/
chown -R prometheus:prometheus /etc/prometheus
chmod -R 775 /etc/prometheus/
chown -R prometheus:prometheus /var/lib/prometheus/
```

vi /etc/systemd/system/prometheus.service
```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
```




## Other

```
./tsdb ls data

./promtool check config prometheus.yml

./prometheus --help

```

```
rate(node_cpu_seconds_total[5m])
```


```

```






```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file /etc/prometheus/prometheus.yml \
  --storage.tsdb.path /prom_data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.enable-lifecycle


[Install]
WantedBy=multi-user.target
```


## Rules

vi rules.yml
```
groups:
  - name: rules-demo
    rules:
      - record: job:node_cpu_seconds:usage
        expr: sum without(cpu, mode)(rate(node_cpu_seconds_total{mode!="idle"}[5m]))/count without(cpu)(count without(mode)(node_cpu_seconds_total)) * 100
      - alert: CPUUsageAbove20%
        expr: 60 > job:node_cpu_seconds:usage > 20
        for: 1m
        labels:
          severity: warn
          team: dev
      - alert: CPUUsageAbove60%
        expr: job:node_cpu_seconds:usage >= 60
        for: 1m
        labels:
          severity: urgent
          team: devops, management

  - name: alert_rules
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
            severity: critical
        annotations:
            summary: 'Instance {{ $labels.instance }} down'
            description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute.'

    - alert: DiskSpaceFree10Percent
        expr: node_filesystem_free_percent <= 10
        labels:
            severity: warning
        annotations:
            summary: 'Instance {{ $labels.instance }} has 10% or less Free disk space'
            description: '{{ $labels.instance }} has only {{ $value }}% or less free.'
```

```
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="myjob"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency
```

```
groups:
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```


Good Examples:  
- https://samber.github.io/awesome-prometheus-alerts/rules.html
- https://samber.github.io/awesome-prometheus-alerts/
- https://sysdig.com/blog/prometheus-alertmanager/
- https://www.squadcast.com/blog/prometheus-sample-alert-rules
- https://github.com/samber/awesome-prometheus-alerts


vi prometheus.yml
```
rule_files:
  - rules/rules*.yml
```


## Prometheus Config

```
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - rules/rules*.yml
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "alertmanager"
    static_configs:
      - targets: ["localhost:9093"]

  - job_name: "node_exporter"
    static_configs:
      - targets:
        - "172.16.0.226:9100"
        - "172.16.0.168:9100"
        - "172.16.0.130:9100"
        - "172.16.0.192:9100"
        - "172.16.0.73:9100"
        - "172.16.0.20:9100"
        - "172.16.0.45:9100"
```


