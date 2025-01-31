Um Metriken vom Blackbox Exporter über HTTPS an Prometheus unter Red Hat 9 zu übergeben, gehst du folgendermaßen vor:

1️⃣ Blackbox Exporter installieren
Falls noch nicht installiert:

```
sudo dnf install -y wget tar
cd /opt
wget https://github.com/prometheus/blackbox_exporter/releases/latest/download/blackbox_exporter-*.linux-amd64.tar.gz
tar -xvzf blackbox_exporter-*.linux-amd64.tar.gz
cd blackbox_exporter-*
sudo mv blackbox_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false blackbox
sudo chown blackbox:blackbox /usr/local/bin/blackbox_exporter
```


2️⃣ Blackbox Exporter konfigurieren
Erstelle oder bearbeite die Datei /etc/blackbox/blackbox.yml:

```
modules:
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_http_versions: [ "HTTP/1.1", "HTTP/2" ]
      method: GET
      tls_config:
        insecure_skip_verify: false  # Falls du ein eigenes Zertifikat nutzt
```
Falls dein Zielserver ein selbstsigniertes Zertifikat hat, setze insecure_skip_verify: true.

3️⃣ Blackbox Exporter als Service einrichten
Erstelle die Datei /etc/systemd/system/blackbox_exporter.service:

```
[Unit]
Description=Blackbox Exporter
After=network.target

[Service]
User=blackbox
Group=blackbox
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox/blackbox.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

Dann:

```
sudo systemctl daemon-reload
sudo systemctl enable blackbox_exporter --now
sudo systemctl status blackbox_exporter
```


4️⃣ Prometheus konfigurieren
Bearbeite die Datei /etc/prometheus/prometheus.yml:

```
scrape_configs:
  - job_name: "blackbox"
    metrics_path: /probe
    params:
      module: [http_2xx]  # Blackbox-Modul verwenden
    static_configs:
      - targets:
        - "https://dein-zielserver.com"
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: "127.0.0.1:9115"  # Blackbox Exporter Endpoint
```


Lade Prometheus neu:

```
sudo systemctl restart prometheus
```


5️⃣ Testen
Teste den Exporter:

```
curl -X GET "http://localhost:9115/probe?target=https://dein-zielserver.com&module=http_2xx"
```


Falls Prometheus korrekt zieht:

```
curl -X GET "http://localhost:9090/api/v1/query?query=probe_success"
```

Falls du HTTPS für Prometheus selbst nutzen möchtest, brauchst du ein Zertifikat für Prometheus. Das kann z. B. mit Let's Encrypt oder einem internen Zertifikat gemacht werden.




## Zertifikat für Blackbox Exporter erstellen

```
mkdir -p /etc/blackbox/tls
cd /etc/blackbox/tls

# Private Key erstellen
openssl genpkey -algorithm RSA -out blackbox.key

# Zertifikatsanfrage erstellen
openssl req -new -key blackbox.key -out blackbox.csr -subj "/CN=blackbox.local"

# Self-Signed Zertifikat erstellen (gültig für 365 Tage)
openssl x509 -req -in blackbox.csr -signkey blackbox.key -out blackbox.crt -days 365


```


Zertifikat in den Blackbox Exporter einbinden


Öffne die Datei /etc/systemd/system/blackbox_exporter.service und ändere den Startbefehl:
```
[Service]
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox/blackbox.yml --web.config.file=/etc/blackbox/tls/web-config.yml

```

Erstelle die Datei /etc/blackbox/tls/web-config.yml:
```
tls_server_config:
  cert_file: "/etc/blackbox/tls/blackbox.crt"
  key_file: "/etc/blackbox/tls/blackbox.key"

```


```
sudo chown -R blackbox:blackbox /etc/blackbox/tls
sudo chmod 600 /etc/blackbox/tls/blackbox.key

```

```
sudo systemctl daemon-reload
sudo systemctl restart blackbox_exporter

```



## Zertifikat für Prometheus erstellen

```
mkdir -p /etc/prometheus/tls
cd /etc/prometheus/tls

# Private Key erstellen
openssl genpkey -algorithm RSA -out prometheus.key

# Zertifikatsanfrage erstellen
openssl req -new -key prometheus.key -out prometheus.csr -subj "/CN=prometheus.local"

# Self-Signed Zertifikat erstellen (gültig für 365 Tage)
openssl x509 -req -in prometheus.csr -signkey prometheus.key -out prometheus.crt -days 365

```


## Prometheus mit TLS konfigurieren

Bearbeite die Datei /etc/systemd/system/prometheus.service, falls vorhanden:
```
[Service]
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --web.config.file=/etc/prometheus/tls/web-config.yml

```

Erstelle die Datei /etc/prometheus/tls/web-config.yml:
```
tls_server_config:
  cert_file: "/etc/prometheus/tls/prometheus.crt"
  key_file: "/etc/prometheus/tls/prometheus.key"

```



```
sudo chown -R prometheus:prometheus /etc/prometheus/tls
sudo chmod 600 /etc/prometheus/tls/prometheus.key

sudo systemctl daemon-reload
sudo systemctl restart prometheus

```



Prometheus mit HTTPS zu Blackbox verbinden

Bearbeite /etc/prometheus/prometheus.yml:
```
scrape_configs:
  - job_name: "blackbox"
    metrics_path: /probe
    scheme: https  # Jetzt HTTPS!
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - "https://dein-zielserver.com"
    tls_config:
      ca_file: "/etc/blackbox/tls/blackbox.crt"
      insecure_skip_verify: false  # Falls du ein selbstsigniertes Zertifikat nutzt
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: "127.0.0.1:9115"  # Blackbox Exporter über HTTPS

```


```
sudo systemctl restart prometheus
```




## Teste Blackbox über HTTPS

```
curl -k https://localhost:9115/probe?target=https://dein-zielserver.com&module=http_2xx

curl -k https://localhost:9090/api/v1/query?query=probe_success

sudo journalctl -u blackbox_exporter -f
sudo journalctl -u prometheus -f

```



ChatGPT kann Fehler machen. Ü

