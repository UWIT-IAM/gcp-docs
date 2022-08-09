# Helm Chart & Release Maintenance

Our applications are now largely reliant on a shared helm chart,
which makes our clusters easier to maintain.

Most applications have a `helm-release.yaml` which describes the values that 
are passed into the chart itself.

When filling out new charts, have a look at the [default values] and their
documentation to make sure you are defining something that meets your needs.

You can find more information also in the [helm chart README] regarding how to 
update the helm chart.

[default values]: https://github.com/uw-iti-app-platform/helm-charts/tree/main/basic-web-service/values.yaml
[helm chart README]: :https://github.com/uw-iti-app-platform/helm-charts/tree/main/basic-web-service/README.md

If you want to update an application to use a newer chart version, 
simply look for the configuration block in your `HelmRelease` that looks like:

```
chart:
spec:
  chart: basic-web-service
  version: 0.3.6
  sourceRef:
    kind: HelmRepository
    name: helm-chart-repository
    namespace: default
```

and update the `version` field to the version you want. When your change is merged 
into the `main` branch of `gcp-k8`, then your chart reconciliation will begin.
