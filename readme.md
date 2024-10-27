# Promethus

In this repository, we **won't discuss all of the Prometheus configuration options and monitor system scheme.** It only focuses on how the configuration leads Prometheus to scrape targets.

## How prometheus know target that will be scrape

### Know how it work

You can access `http://<prometheus_host:port>/api/v1/targets?state=active`

There are a lot of scrape target in the `data.activeTargets` 

There will give a static target 

```yaml
- job_name: OA_group
  honor_timestamps: true
  scrape_interval: 30s
  scrape_timeout: 30s
  metrics_path: /metrics
  scheme: http
  static_configs:
  - targets:
    - 10.130.147.129:9100
  relabel_configs:
  - source_labels: [__address__]
    separator: ;
    regex: ([0-9a-zA-Z\\.]+):[0-9]+
    target_label: instance
    replacement: $1
    action: replace
```

Then, when you access the API URL, it will display the JSON data

>```json
>{
> "discoveredLabels": {
>     "__address__": "10.130.147.129:9100", 
>     "__metrics_path__": "/metrics", # This is built-in variables those is url of scarp target
>     "__scheme__": "http", # This is built-in variables those is scheme of scarp target
>     "job": "OA_group"
> },
> "labels": {
>     "instance": "10.130.147.129",
>     "job": "OA_group"
> },
> "scrapePool": "OA_group",
> "scrapeUrl": "http://10.130.147.129:9100/metrics",
> "globalUrl": "http://10.130.147.129:9100/metrics",
> "lastError": "",
> "lastScrape": "XXXXXXXXXXXX",
> "lastScrapeDuration": 0.03377565,
> "health": "up"
>}
>```

let's see those value

1. `discoveredLabels.__scheme__:http `
   1.  this value is from config. that Yaml is `scheme` 
   2.  scarp url will become `http`
2. `discoveredLabels.__address__: 10.130.147.129:9100 ` 
   1. this value is from config. that Yaml is `static_configs.[X].targets.[X]`
   2. scarp url will  become  `http://10.130.147.129:9100`

3. `discoveredLabels.__metrics_path__: /metrics`
   1. this value is from config. that Yaml is `metrics_path`
   2. scarp url will  become  `http://10.130.147.129:9100/metrics`

4. `discoveredLabels.__param_module: http_2xx`  Sometimes it is not visible because the configuration is not set
   1.  This will be set using a special method. `params.[PARMS_NAME].[X]`
   2.  scarp url will  become  `http://10.130.147.129:9100/metrics?module=http_2xx`


### How to change?

You can use label and `relabel_configs` to set that

```yaml
- job_name: OA_group
  .....
  static_configs:
  - targets:
    - 172.16.1.209
    labels:
      newTarget: 192.168.22.22
```

those will append a new `newTarget="192.168.22.22"` label.

```yaml
- job_name: OA_group
  .....
  static_configs:
  - targets:
    - 10.130.147.129:9100
    labels:
      newTarget: 192.168.22.22:33
  relabel_configs:
  - source_labels: ['newTarget']
    action: replace
    target_label: '__address__'
```

those will set 

`discoveredLabels.__address__ `  from   value  `10.130.147.129:9100`  to  `192.168.22.22:33`

and There you can set scheme

```yaml
- job_name: OA_group
  .....
  static_configs:
  - targets:
    - 10.130.147.129:9100
    labels:
      newTarget: 192.168.22.22:33
      newScheme: https
  relabel_configs:
  - source_labels: ['newTarget']
    action: replace
    target_label: '__address__'
  - source_labels: ['newScheme']
    action: replace
    target_label: '__scheme__'
```

There you can set path

```yaml
- job_name: OA_group
  .....
  static_configs:
  - targets:
    - 10.130.147.129:9100
    labels:
      newTarget: 192.168.22.22:33
      newScheme: https
      newPath: /path
  relabel_configs:
  - source_labels: ['newTarget']
    action: replace
    target_label: '__address__'
  - source_labels: ['newScheme']
    action: replace
    target_label: '__scheme__'
  - source_labels: ['newPath']
    action: replace
    target_label: '__metrics_path__'
```

There you can set pamrs

```yaml
- job_name: OA_group
  .....
  static_configs:
  - targets:
    - 10.130.147.129:9100
    labels:
      newTarget: 192.168.22.22:33
      newScheme: https
      newPath: /path
      newParmsRq: rqqqq
  relabel_configs:
  - source_labels: ['newTarget']
    action: replace
    target_label: '__address__'
  - source_labels: ['newScheme']
    action: replace
    target_label: '__scheme__'
  - source_labels: ['newParmsRq']
    action: replace
    target_label: '__param_Rq'
```

## Authentication

Sometimes, scraping targets require authentication, so you need to set up authentication. like

```yaml
basic_auth
    username: prometheus
    password: scarpspasswor
```

However, authentication cannot be changed using `relabel_configs`. Therefore, if you have many scrape targets that require authentication, you cannot use `relabel_configs` for this purpose. We do not know if this will Improve in the future.
