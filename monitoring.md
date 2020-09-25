# Application Monitoring with Prometheus/Grafana/Alerts

Required Reading: [MCI Documentation on Prometheus/Grafana/Alerts](https://wiki.cac.washington.edu/display/MCI/Monitoring+and+Alerting#MonitoringandAlerting-LocalConventionsforAlertsinMCI)

In identityuw metrics are exposed via the `/metrics` endpoint, which prometheus
scrapes. With that in place, you can view metrics results from grafana or
prometheus itself. Using the prometheus console is good for constructing your
queries. After you've done that you can drop them into the more-friendly Grafana
UI, or use them as a threshold to trigger alerts. See
[Querying Prometheus](https://prometheus.io/docs/prometheus/latest/querying/basics/)
for details. An example query from [identityuw](https://github.com/UWIT-IAM/gcp-k8/tree/master/prod/identityuw/prometheus) is
`sum(increase(identityuw_high_loglevel_total{level="ERROR"}[2m]))`. In this
example, we take the total number of errors logged over two minutes.

## Viewing metrics

Generated metrics are best viewed from Grafana. The 
[Identity.UW metrics dashboard](https://uwiam.page.link/grafana) shows a set of
metrics that capture overall system health. For now the dashboard needs to be loaded
by hand when the grafana pod restarts. See
[the grafana config](https://github.com/UWIT-IAM/gcp-k8/tree/master/etc) for details.

## Generating an alert

With this example, again from [identityuw](https://github.com/UWIT-IAM/gcp-k8/tree/master/prod/identityuw/prometheus)...

```yaml
- alert: application_errors
  expr: 'sum(increase(identityuw_high_loglevel_total{level="ERROR"}[2m])) > 0'
  labels:
    ci_name: "Identity.UW website"
    alert_notify: "true"
    focus: "5"
  annotations:
    summary: Identity.uw has had a server error
    description: View the current logs in Stackdriver with https://uwiam.page.link/errors.
```

We say that if the number of errors logged is ever greater than zero, then create
an alert. `focus: "5"` tells it to send a slack message to channel `#iam-infra-prod`.
A 4 or higher should create an incident in ServiceNow. You could create a second
alert that does this, maybe with a threshold of 10 instead of 0.
