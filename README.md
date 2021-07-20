# prometheus-blackbox-exporter
https://github.com/prometheus/blackbox_exporter

## Local test

```sh
podman run --rm -i -p 9115:9115 --name blackbox-exporter -v `pwd`/config:/config:Z quay.io/prometheus/blackbox-exporter:latest --config.file=/config/blackbox-oauth2.yaml
```

```sh
curl "http://localhost:9115/probe?target=https://google.com&module=http_2xx"
```
## Deploy

```sh
helm upgrade -i prometheus-blackbox-exporter helm/prometheus-blackbox-exporter -n prometheus-blackbox-exporter --create-namespace
```

## Prometheus Scrape Configs

> TODO

```yaml
scrape_configs:
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - https://prometheus.io   # Target to probe with https.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115  # The blackbox exporter's real hostname:port.
```
