# New Image Pull Secret for GKE to GCR

By default, GKE has read permissions on the GCR storage bucket in the same GCP Project as the cluster.  That's not what we have.  We have our GCR in the `uwit-mci-iam` project and GKE is a terraform created projects.

## NOTE

This entire readme must be done once per cluster.

- [x] `dev` cluster.
- [x] `eval` cluster.
- [x] `prod` cluster.

## Prerequisite

1. You have configured the [iamshared gcloud CLI profile](projects-shared.md) and have access to that project.

2. You have configured the `[k8dev gcloud CLI profile]` and can use it.

    ```
    gcloud config configurations activate k8dev
    ```

## Service Account Setup

**This is already done for `uwit-mci-iam`.** Do this once per project that is hosting a GCR.

The purpose of this account is limited to GCR read in GCP projects external from `uwit-mci-iam`.

1. Use the `uwit-mci-iam` shared config.

    ```
    gcloud config configurations activate iamshared
    ```

1. Create the service account.

    ```
    gcloud iam service-accounts create k8-gcr --display-name "k8 cluster imagePullSecret"
    ```

1. Grant the reader permissions for the bucket, which one of these is correct?

    ```
    gsutil iam ch serviceAccount:k8-gcr@uwit-mci-iam.iam.gserviceaccount.com:objectViewer gs://artifacts.uwit-mci-iam.appspot.com

    gcloud projects add-iam-policy-binding uwit-mci-iam --member serviceAccount:k8-gcr@uwit-mci-iam.iam.gserviceaccount.com  --role roles/storage.objectViewer
    ```

1. Create a new key for this new SA

    ```
    gcloud iam service-accounts keys create ./key.json --iam-account k8-gcr@uwit-mci-iam.iam.gserviceaccount.com
    ```

## Kubernets Setup

Do this onece per kubernetes cluster that will pull from GCR.

1. Create the k8 secret that all deployments will use and refrence in their `imagePullSecrets` yaml.  The only special value below is the `key.json` which should be the result from doing the steps above.

    ```
    kubectl create secret docker-registry gcr-reader \
      --docker-username=_json_key \
      --docker-password="$(cat key.json)" \
      --docker-server=https://gcr.io \
      --docker-email=depricated@depricated.com
    ```

1. Attach the secret to the Kubernetes Service Account that is used by your pods.  This step below assumes your pod is using the `default` service account.  This is a one shot trick instead of attaching the secret to every single deployment or pod spec.  As far as app developers are concerned, it just works, nothing else to do.

    ```
    kubectl get serviceaccounts default -o json | jq 'del(.metadata.resourceVersion)' | jq 'setpath(["imagePullSecrets"];[{"name":"gcr-reader"}])' | kubectl replace serviceaccount default -f -
    ```