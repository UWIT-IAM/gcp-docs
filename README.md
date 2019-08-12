# UWIT-IAM GCP Technical Documentation

This is a public repo, do not store service account names, project id's or other sensitive configuration details.

## Web Applications

- [How do I configure my gcloud CLI](new-gcloud-profile.md)
- [How do I create a new application and have it pushed to GCR](new-application.md)
- [How do I manually deploy my application to the dev cluster](new-deployment.md)
- [How do I auto deploy my application to a cluster](https://github.com/UWIT-IAM/gcp-k8)
- [How do I serve HTTP/HTTPS traffic to/from my application](edit-ingress.md)
- [How do I serve static content js/css/html from a CDN](edit-cdn.md)

## Application Administration

- [How do I view logs/stdout](get-logs.md)
- [Application Monitoring with Prometheus/Grafana/Alerts](monitoring.md)

## Cluster Administration

- [How do I get DNS resolving to a GKE cluster](new-hostedzone.md)
- [How do I enable TLS traffic into a GKE cluster](new-ingress.md)
- [How do I enable a cluster to pull from GCR](new-imagepullsecret.md)
- [How do I create a Load Balancer for CDN based buckets](new-cdn.md)
- [Create or edit secrets](secrets.md)
- The naming conventions in use can be found by looking at the `/examples` dirctory.

### New Cluster Setup

Google Cloud Projects and GKE clusters are created by the UE team using Terraform.  They are all in a GCP Shared VPC.  Once they are created we are responsible with the workloads inside the cluster.

1. Get a cluster provisioned from UE and have cluster admin access.
1. Get a simple basic [new application](new-application.md) running.
1. Create a [hosted zone](new-hostedzone.md)
1. Create a [TLS Ingress Service](new-ingress.md)
1. Enable the default k8 service account to [pull from GCR](new-imagepullsecret.md)


## Contributing

1. If you see an error in this repo, clone it, commit, make a PR
1. Do not put sensitive information in this public repo, instead, provide commands that enable the discovery of service accounts or project id's.
1. Most "setup" tasks are already done and this provides a history of those one time actions, which, ideally could be automated via terraform etc.
