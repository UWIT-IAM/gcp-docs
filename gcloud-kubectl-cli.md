# New gcloud CLI Profile

Ideally you are automating all the things and are fortunate enough to not need to use `gcloud` or `kubectl` commands.  
If you do need these commands use this guide as the rest of the documentation in this repository assumes you have done 
these steps here.

Your access is granted to GCP via UW Groups membership within the 
[u_mciman_tenants_iam_operators](https://groups.uw.edu/search/?name=u_mciman_tenants_iam&stem=&member=&owner=&type=effective&scope=one) group.

# VPN Connectivity

Because our kubernetes clusters are in a VPC network that only offers connectivity to the UW network, you have two 
options for connecting to our cluster:

* Use [Husky OnNet](https://itconnect.uw.edu/connect/uw-networks/about-husky-onnet/) 
in "All Internet Traffic" mode; use your laptop to connect to the cluster directly.
* Use Husky OnNet in "UW Internet Traffic Only" mode; connect to an IAM bastion (or other on-network device that 
accepts 2FA SSH); use your SSH connection to connect to the cluster.

# Switching Clusters

Make sure you know which cluster you are working with before you do anything! Check first with:

```bash
gcloud config get-value project
```

See the [project documentation](https://wiki.cac.washington.edu/pages/viewpage.action?pageId=125261222) to decode 
the project id.

Switching to a different cluster is a two-step process. 

First, switch your gcloud project:

```bash
gcloud config set project "$PROJECT_ID"
```

Then, you have to have gcloud rebuild your kubeconfig profile:

```bash
gcloud container clusters get-credentials "${PROJECT_ID}-cluster" --region us-west1
```
