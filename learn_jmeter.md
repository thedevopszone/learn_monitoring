# JMeter

## Install

```
yum install java-11-openjdk-devel
java --version
```

```
wget https://downloads.apache.org//jmeter/binaries/apache-jmeter-5.4.1.tgz
```

```
tar -xvzf apache-jmeter-5.4.1.tgz
```

vi /etc/systemd/system/jmeter.service
```
[Unit]
Description=Apache JMeter Service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/opt/jmeter/bin/jmeter -n -t /path/to/your/testplan.jmx -l /path/to/your/results.jtl
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl daemon-reload
sudo systemctl start jmeter
sudo systemctl enable jmeter
sudo systemctl status jmeter
```

