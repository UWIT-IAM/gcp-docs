# Viewing Logs

Logs are sent to Stackdriver, but, can be fetched at the command line using `kubectl` or with the Stackdriver UI.

## Prerequisite

1. You also have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md).

1. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## Stackdriver K8 Lable

If you add labels to your deployment or service YML you can make querying for anything and everything that has that label and easy process.  By putting the following filter into Stackdriver you can query for the `identityuw` label.

    metadata.userLabels.app="identityuw"

You can then get the link to that filter and it will look something like this.  In that link the `authuser=1` QS param may need to change for you if you have multiple Google accounts.


https://console.cloud.google.com/logs/viewer?interval=P1D&project=uwit-mci-0003&authuser=1&organizationId=657476903663&minLogLevel=0&expandAll=false&customFacets&limitCustomFacetWidth=true&advancedFilter=metadata.userLabels.app%3D%22identityuw%22&scrollTimestamp=2018-12-17T17%3A57%3A45.000000000Z&dateRangeStart=2018-12-16T17%3A58%3A32.578Z&dateRangeEnd=2018-12-17T17%3A58%3A32.578Z


## kubectl Pods

There are a [few more things](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#interacting-with-running-pods) you can do with logs than whats listed here.

1. Find your pod that you want logs of

    ```
    kubectl get pods
    ```

1. View the logs...

    ```
    kubectl logs [podname] image
    ```

1. ...or stream the logs

    ```
    kubectl logs -f [podname]
    ```

1. Remember, the logs you see are for the currently running deployment.  You will have to get the pods of a previous deployment to see those.