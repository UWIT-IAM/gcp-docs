# New K8 Deployment
The goal of this guide is to provide the manual steps needed to getting a K8 deployment and it's service running in our dev cluster.  This is mostly taken from the GKE Quickstart.

This is a manual process and not to be used for [automated deployments](new-cicd.md).


## Prerequisite
1. You have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md) correctly.
2. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## Create Your Deployment

