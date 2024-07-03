# Learn Fluent-bit

## Install

```
brew install fluent-bit

git clone https://github.com/grafana/loki.git
cd  loki
make fluent-bit-plugin
```

## Config

vi fluent-bit.conf
```
[INPUT]
  Name    tail
  Path    /var/log/*.log
  Path_Key    filename

[Output]
  Name  loki
  Match  *
  Url  http://localhost:3100/loki/api/v1/push
  BatchWait    1
  BatchSize    3104304
  Labels      {job="varlogs",host="ward_workstation",agent="fluent-bit"}
  LabelKeys   filename
```


## Run

```
fluent-bit -e loki/cmd/fluent-bit/out_loki.so -c fluent-bit.conf
```



