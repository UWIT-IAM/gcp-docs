# New Image Pull Secret for GKE to GCR

The Google Cloud Project that hosts our GCR storage bucket is not the same Project that GKE resides in, therefore, GKE does not have any read permissions to the bucket and this guide details how to set that up.

## NOTE

This entire readme must be done once per cluster.

- [x] `dev` cluster.
- [x] `eval` cluster.
- [x] `prod` cluster.

## Prerequisite

1. You have configured the `[k8dev gcloud CLI profile]` and can use it.

1. You have configured the [iamshared gcloud CLI profile](projects-shared.md) and have access to that project.

    ```
    gcloud config configurations activate iamshared
    ```

## Service Account Setup

Do this once per project that is hosting a GCR.  This will create Google Cloud Service Account to be used within the GKE clusters.

1. Use the GCP shared project config.

    ```
    gcloud config configurations activate iamshared
    ```

1. Create the service account.  In this example it's name is `kuber-gcr`

    ```
    gcloud iam service-accounts create kuber-gcr --display-name "k8 cluster imagePullSecret"
    ```

1. Get the email of the service account, also get the bucket url used for our image registry.  It will be in the format of `gs://artifacts.[project id].appspot.com`.

   ```
   gcloud iam service-accounts list
   gsutil ls
   ```

1. Grant the reader permissions for the bucket, which one of these is correct?

    ```
    gsutil iam ch serviceAccount:[service account email]:objectViewer [bucket url]

    gcloud projects add-iam-policy-binding [project id] --member serviceAccount:[service account email]  --role roles/storage.objectViewer
    ```

1. Create a new key for this new SA

    ```
    gcloud iam service-accounts keys create ./key.json --iam-account [service account email]
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