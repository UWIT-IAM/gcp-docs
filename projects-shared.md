# Goolge Cloud Projects - Shared

This project is meant to provide shared services for all our GCP projects.  Things like DNS (Cloud DNS), Container Registry (GCR) and more are located here.  The changes you do here impact production and access should be limited.

## Setup

1. You have direct GCP IAM Access to the `uwit-mci-iam` project (restricted UWIT-IAM cloud superusers)

1. Create a new gcloud profile called `iamshared`.  Use the [instructions for the k8dev profile](new-glcoud-profile.md).

1. You are using the iamtools profile.

    ```
    gcloud config configurations activate iamshared
    ```

1. Confirm your access `gcloud dns managed-zones list`