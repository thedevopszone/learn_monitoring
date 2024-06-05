

## Install

```
yum install java-11-openjdk java-11-openjdk-devel

alternatives --config java
```

vi /etc/systemd/system/zipkin.service
```
# Zipkin System Service
[Unit]
Description=Manage Java service
Documentation=https://zipkin.io/

[Service]
WorkingDirectory=/opt/zipkin
ExecStart=/usr/bin/java -jar zipkin.jar
User=zipkin
Group=zipkin
Type=simple
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```
