# prometheus-blackbox-exporter

See <https://github.com/prometheus/blackbox_exporter>.

This is an example to use the blackbox exporter with oauth2 from Okta. You will need to configure a proxy that requires oauth2 in front of the service, for example: <https://github.com/trevorbox/service-mesh-patterns/tree/master/ossm-2.0/auth>

## Local test

```sh
podman run --rm -i -p 9115:9115 --name blackbox-exporter -v `pwd`/config:/config:Z quay.io/prometheus/blackbox-exporter:latest --config.file=/config/blackbox-oauth2.yaml
```

## Test from blackbox pod

> Note this won't work because oauth is enabled

```sh
curl "http://localhost:9115/probe?target=https://google.com&module=http_2xx"
```
## Deploy

```sh
helm upgrade -i prometheus-blackbox-exporter helm/prometheus-blackbox-exporter -n sre-monitoring --create-namespace \
  --set okta.client_id=<my_client_id> \
  --set okta.client_secret=<my_client_secret> \
  --set okta.token_url=<my_token_url> # example: https://dev-338970.okta.com/oauth2/default/v1/token
```

## Prometheus Scrape Configs

Example scrape configs for Prometheus...

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
        replacement: prometheus-blackbox-exporter:9115  # The blackbox exporter's real hostname:port (matches the service name)
```
