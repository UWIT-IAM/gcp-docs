# Viewing Logs

Logs are sent to Stackdriver, but, can be fetched at the command line using `kubectl` or with the Stackdriver UI.

## Prerequisite

1. You have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md).

1. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## Stackdriver K8 Label

If you add labels to your deployment or service YML you can make querying for anything and everything that has that label and easy process.  By putting the following filter into Stackdriver you can query for the `identityuw` label.

    metadata.userLabels.app="identityuw"

## Stackdriver + CLI
Using the google cloud `gcloud` cli you can leverage the same filter that you create in the cloud console to query logs in a terminal.  The following example gets all log entries for the identityuw container within a time range, but, exclude log entries with `static/vendor` in the payload.

This then pipes to `jq` to do further processing as without it you will get the same verbose output as the cloud console.  See [JQ play](https://jqplay.org/) and documentation for more.

Refer to the [google cloud documentation](https://cloud.google.com/logging/docs/view/advanced-filters) on how to create and use filters.

```
gcloud logging --format=json read '/
  resource.labels.container_name=identityuw /
  resource.type=k8s_container /
  timestamp >= "2019-01-29T09:00:00-00:00" /
  timestamp <= "2019-01-29T14:30:00-00:00" /
  NOT textPayload:"static/vendor"' | jq -rj '.[].timestamp + " " + .[].textPayload'
```

Produces output similar to the folloiwng, the extra dates are there becuase at this time that app was sending it's own time stamps as well (not required).
```
2019-01-29T11:32:33.424280399Z Not Found: /m.php?pbid=open
2019-01-29T11:32:33.424107693Z Not Found: /m.php?pbid=open
2019-01-29T11:32:33.424280399Z 2019-01-29 03:32:33,423 WARNING base.get_response():93: Not Found: /m.php?pbid=open
2019-01-29T11:32:33.424107693Z 2019-01-29 03:32:33,423 WARNING base.get_response():93: Not Found: /m.php?pbid=open
2019-01-29T11:32:33.424280399Z Not Found: /.well-known/security.txt
2019-01-29T11:32:33.424107693Z Not Found: /.well-known/security.txt
2019-01-29T11:32:33.424280399Z 2019-01-29 03:26:44,023 WARNING base.get_response():93: Not Found: /.well-known/security.txt
2019-01-29T11:32:33.424107693Z 2019-01-29 03:26:44,023 WARNING base.get_response():93: Not Found: /.well-known/security.txt
2019-01-29T11:32:33.424107693Z Not Found: /static/CACHE/css/cb8e8d41a7ed.css
2019-01-29T11:32:33.424280399Z Not Found: /db_pma.php
2019-01-29T11:32:33.424107693Z Not Found: /db_pma.php
2019-01-29T10:04:23.156309005Z Not Found: /db_pma.php
2019-01-29T11:32:33.424280399Z 2019-01-29 02:04:23,156 WARNING base.get_response():93: Not Found: /db_pma.php
2019-01-29T11:32:33.424107693Z 2019-01-29 02:04:23,156 WARNING base.get_response():93: Not Found: /db_pma.php

```


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