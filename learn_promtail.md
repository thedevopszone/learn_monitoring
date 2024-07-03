# Learn Promtail


## Config

vi config.yml
```
scrape_configs:
  - job_name: system
    pipeline_stages:
    static_configs:
    - targets:
        - localhost
        labels:
          job: varlogs
          host: ward_workstation
          agent: promtail
          __path__: /var/log/*.log
```

## Run Promtail

```
./promtail -client.url http://localhost:3100/loki/api/v1/push -config.file=config.yml -positions.file=positions.yml


```



