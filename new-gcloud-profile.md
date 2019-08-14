# New GCloud CLI Profile

Ideally you are automating all the things and are fortunate enough to not need to use `gcloud` or `kubectl` commands.  If you do need these commands use this guide as the rest of this repo assumes you have done these steps here.

## Prerequisite

1. You understand how access is granted to GCP via UW Groups membership within the [u_mciman_tenants_iam_operators](https://groups.uw.edu/search/?name=u_mciman_tenants_iam&stem=&member=&owner=&type=effective&scope=one) group.

1. You have a [local shell setup](https://cloud.google.com/sdk/docs/quickstarts) for `gcloud` and **authenticated it with your netid**.

1. You have ran `gcloud components install kubectl`.

1. You can run the following command, which tells us you have permissions to the right project with the dev k8 cluster.

    ```BASH
    gcloud projects list --filter="labels.environment:dev"
    ```

1. You are on a UW Network, all kubernetes API interactions will require this.  Use Husky OnNet if needed.

## Create a gcloud Profile

The `gcloud` command can execute commands against any number of Google accounts, projects, regions and zones.  To make our life eaiser and more automated lets create a profile that is specific to executing commands against the development Google Cloud Project and Kubernetes cluster.

1. Run the following command, the value returned will now be used as `[PROJECT-ID]`.  Use `eval` or `prod` instead of `dev` below if creating profiles for those environments.

    ```BASH
    gcloud projects list --filter="labels.environment:dev"
    ```

1. Create a new glcoud config by typing `gcloud init`...
   - Select "Create a new configuration"
   - Give it the name `k8dev` **required, this repo assumes this name**
   - Choose log in with new account, or, your netid if it is already listed
   - Select the `[PROJECT-ID]` from the pre-reqs.

1. Get the GCP region and zone that the GKE cluster is in.

    ```BASH
    gcloud container clusters list --project=[PROJECT-ID]
    ```

1. Set the region and zone to match the output from the previous step.  Region will be under `LOCATION`.  Region being `us-west1` and zone being `us-west1-c` for example.

    ```BASH
    gcloud config set compute/region [region]
    gcloud config set compute/zone [zone]
    ```

## Confirm Your Access

1. Make sure your gcloud access is setup correctly, you should not get errors from the following commands.

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
    kubectl get pods
    ```

1. Rename the `kubectl` context so that you can manage multiple k8 clusters more eaisly.  Switching `gcloud config configurations` profiles does not automaticly switch `kubectl config current-context`, they are managed independantly.

   ```BASH
   kubectl config get-contexts
   kubectl config rename-context [name from prev cmd] [shorter name]
   kubectl config use-context [shorter name]
   ```
