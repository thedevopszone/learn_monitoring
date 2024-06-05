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
```


vi prometheus.yml
```
rule_files:
  - rules/rules*.yml
```



