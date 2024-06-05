# Alertmanager

## Install

```
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz

xvzf alertmanager-0.27.0.linux-amd64.tar.gz

cd alertmanager-0.27.0.linux-amd64

mkdir /etc/alertmanager
cp alertmanager.yml /etc/alertmanager/

useradd --no-create-home --shell /bin/false alertmanager

cp alertmanager /usr/local/bin/
cp amtool /usr/local/bin/

chown alertmanager:alertmanager /usr/local/bin/alertmanager
chown alertmanager:alertmanager /usr/local/bin/amtool
chown -R  alertmanager:alertmanager /etc/alertmanager
```

vi /etc/systemd/system/alertmanager.service
```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
WorkingDirectory=/etc/alertmanager/
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --web.external-url http://192.168.122.80:9093

[Install]
WantedBy=multi-user.target
```

vi etc/prometheus/prometheus.yml
```
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['localhost:9093']

scrape_configs:
  ...

  - job_name: alertmanager
    static_configs:
      - targets: ['localhost:9093']
```

Check config
```
promtool check config /etc/prometheus/prometheus.yml
```

