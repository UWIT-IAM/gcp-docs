# Viewing Logs
Logs are sent to Stackdriver, but, can be fetched at the command line using `kubectl`.

## Prerequisite
1. You also have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md).
1. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## Pods
There are a [few more things](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) you can do with logs than whats listed here.

1. Find your pod that you want logs of

    ```
    kubectl get pods
    ```

1. View the logs...

    ```
    kubectl logs [podname] image
    ```

1. ...or Stream the logs

    ```
    kubectl logs -f [podname]
    ```

1. Remember, the logs you see are for the currently running deployment.  You will have to get the pods of a previous deployment 