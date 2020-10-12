# Application Monitoring with Prometheus/Grafana/Alerts

You can find the links to our prometheus and grafana dashboards on our [software components wiki](https://wiki.cac.washington.edu/display/SMW/Identity.UW+Software+Components).

Required Reading: [MCI Documentation on Prometheus/Grafana/Alerts](https://wiki.cac.washington.edu/display/MCI/Monitoring+and+Alerting#MonitoringandAlerting-LocalConventionsforAlertsinMCI)

In identity.UW, metrics are exposed via the `/metrics` endpoint, which prometheus
scrapes. 

You can view metrics results from grafana or prometheus itself. 

Using the prometheus console is good for constructing your queries. The console also lists all available metrics for you to choose from. After you've tested your query, you can add it to the Grafana UI for sharing or dashboarding. Queries can also be used to triggert alerts to our slack channel, or create incident in Service Now.

See [Prometheus query documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/) for details. 


An example query from [identityuw](https://github.com/UWIT-IAM/gcp-k8/tree/master/dev/identity-uw/prometheus.yml) is the `unqualified_application_error` which is queried as: `sum(increase(identityuw_log_qualifier_total{level="ERROR",QUALIFIER="unqualified"}[2m]))`. In this
example, we take the total number of unqualified (i.e., unexpected) errors logged over two minutes. (For more information on log qualifiers, see below.)


## Generating an alert

With this example, again from [identity.UW](https://github.com/UWIT-IAM/gcp-k8/blob/master/dev/identity-uw/prometheus.yml):

```yaml
- alert: unqualified_application_errors
      expr: 'sum(increase(identityuw_log_qualifier_total{level="ERROR",qualifier="unqualified"}[2m])) > 0'
      labels:
         ci_name: "Identity.UW website"
         alert_notify: "true"
         focus: "5"
      annotations:
        summary: Identity.UW has an increased rate of unexpected errors
        description: View the current logs in Stackdriver with https://uwiam.page.link/errors-dev.
```

* **expr** is the Prometheus query.
* **labels[ci_name]** Refers to our the application configuration item (CI).
* **labels[alert_notify]** Should always be true, unless you awnt to temporarily turn an alert off without deleting its configuration
* **labels[focus]** Should be a number between 1 and 5, where 1–4 will trigger Service Now incidents of an appropriate severity, and 5 will simply send the message to the appropriate slack channel.

The above configuration, therefore, says "If the number of unqualified error over two minutes is greater than 0, send a message to our slack channel and link us to the error logs."
