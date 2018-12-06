# New GCloud CLI Profile

This is a requirement, and, this entire repo assumes you have completed this guide.

## Prerequisite

1. You understand how access is granted to GCP via UW Groups membership within the [u_mciman_tenants_iam_condevs](https://groups.uw.edu/search/?name=u_mciman_tenants_iam&stem=&member=&owner=&type=effective&scope=one) group.

1. You have a local shell setup for [`gcloud`](https://cloud.google.com/sdk/docs/quickstarts) and **authenticated it with your netid**.

1. You have ran the `gcloud components install kubectl`

1. You can run the following command, which tells us you have permissions to the right project with the dev k8 cluster.

    ```BASH
    gcloud projects list --filter="labels.environment:dev"
    ```

1. The output from the command above will now be used as `[PROJECT-ID]` in the rest of this guide.

## Create a gcloud Profile

The `gcloud` command can execute commands against any number of Google accounts, projects, regions and zones.  To make our life eaiser and more automated lets create a profile that is specific to executing commands against the development Google Cloud Project and Kubernetes cluster.

1. Create the new config (this wont change existing ones). You must use `k8dev` as the name, this repo assumes everyone has that profile.

    ```BASH
    gcloud config configurations create k8dev
    ```

1. Initialize this new config by typing `gcloud init`.
   - Select "Re-initialize"
   - Select your `netid@uw.edu`
   - Select the `[PROJECT-ID]` from the pre-reqs.

1. Get the GCP region and zone, we want to set that in our config becuase life is hell when you have to type those for every gcloud command.

    ```BASH
    gcloud container clusters list --project=[PROJECT-ID]
    ```

1. Set the region and zone to match the output from the previous step.

    ```BASH
    gcloud config set compute/region [region]
    gcloud config set compute/zone [zone]
    ```

## Confirm Your Access

1. Make sure your gcloud access is setup correctly, if you followed the pre-reqs you should get a single project from the last command, something like `uwit-mci-0003  iam-dev  303159285527`, though it may not be exactly that.

    ```BASH
    gcloud auth list
    gcloud projects list
    gcloud projects list --filter="labels.environment:dev"
    ```

1. We need to set your `~/.kube/config` credentials so that `kubectl` [commands will work](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials).  If you get prompted for region or zone errors then you did not finish the previous section correctly.

    ```BASH
    gcloud container clusters get-credentials [PROJECT-ID]-cluster
    ```

1. Confirm that `kubectl` works

    ```BASH
    kubectl get all
    ```

1. Rename the `kubectl` context so that you can manage multiple k8 clusters more eaisly.  Switching `gcloud config configurations` profiles does not automaticly switch `kubectl config current-context`, they are managed independantly.

   ```BASH
   kubectl config get-contexts
   kubectl config rename-context [name from prev cmd] [shorter name]
   kubectl config use-context [shorter name]
   ```