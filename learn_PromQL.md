# PromQL

This shows multiple time series results. 1 for each target
```
scrape_duration_seconds
```


we can filter for 1 target by including either the instance, or job labels
```
scrape_duration_seconds{instance="localhost:9100"}
```


Regular Expressions
```
node_cpu_seconds_total{mode=~".*irq"}
```


Range Vector
```
node_netstat_Tcp_InSegs{instance="localhost:9100"}[5m]
node_netstat_Tcp_InSegs{instance="localhost:9100"}[1m]
node_netstat_Tcp_InSegs{instance="localhost:9100"}[30s]
```


Functions
```
rate(scrape_duration_seconds{instance="localhost:9100"}[1m:20s])
```




