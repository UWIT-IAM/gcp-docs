## UWIT-IAM GCP Technical Documentation

### Web Applications
- [How do I create a new application and have it pushed to GCR](new-application.md)
- [How do I configure my gcloud CLI](new-gcloud-profile.md)
- [How do I manually deploy my application to the dev cluster](new-deployment.md)
- [How do I auto deploy my application to a cluster](new-cicd.md)
- [How do I serve HTTP/HTTPS traffic to/from my application](edit-ingress.md)

### Administration
- [How do I get DNS resolving to a GKE cluster](new-hostedzone.md)
- [How do I add/update TLS secrets on an Ingress Service](edit-secrets-tls.md)
- [How do I enable TLS traffic into a GKE cluster](new-ingress.md)
- The naming conventions in use can be found by looking at the [examples](examples/)

#### New Cluster Setup
Google Cloud Projects and GKE clusters are created by the UE team using Terraform.  They are all in a GCP Shared VPC.  Once they are created we are responsible with the workloads inside the cluster.

1. Get a cluster provisioned from UE and have cluster admin access.
1. Get a simple basic [new application](new-application.md) running.
1. Create a [hosted zone](new-hostedzone.md)
1. Create a [TLS Ingress Service](new-ingress.md)