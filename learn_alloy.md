
## Install

```
wget https://github.com/grafana/alloy/releases/download/v1.1.1/alloy-1.1.1-1.amd64.rpm
rpm -i alloy-1.1.1-1.amd64.rpm
```

```
systemctl start alloy
systemctl enable alloy
systemctl status alloy
```

## GUI

```
--server.http.listen-addr=0.0.0.0:12345
```

```
http://localhost:12345
```
