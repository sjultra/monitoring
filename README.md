# Monitoring

One metrics stack per cluster: [VictoriaMetrics](https://victoriametrics.com/) stores, [Grafana](https://grafana.com/) shows. Telemetry apps push to it and add their own dashboards, so several apps share one Grafana.

- **Push metrics** to `http://ingest.<namespace>.svc:8428`: `/write` (InfluxDB line protocol, e.g. Telegraf), `/api/v1/write` (Prometheus remote-write), `/opentelemetry/v1/metrics` (OTLP over HTTP).
- **Add a dashboard** as a ConfigMap labeled `grafana_dashboard: "1"`, in any namespace.
- **Grafana:** `http://<any-node-ip>:30084`, user `admin`; the password is generated into the `grafana` Secret:
  ```
  kubectl -n <namespace> get secret grafana -o jsonpath='{.data.admin-password}' | base64 -d
  ```

## Chart

`helm/service`, an umbrella over the upstream `victoria-metrics-single` and `grafana` charts (pinned in `Chart.lock`).

```
helm repo add victoriametrics https://victoriametrics.github.io/helm-charts/
helm repo add grafana https://grafana.github.io/helm-charts
helm dependency build helm/service
helm install monitoring helm/service --create-namespace --namespace monitoring
```

Published as `oci://ghcr.io/sjultra/charts/monitoring`. To release: Releases → "Draft a new release" → tag `vX.Y.Z` → Publish.
