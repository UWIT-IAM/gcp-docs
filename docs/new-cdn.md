# New Load Balancer for CDN, Buckets and Static Content

In order to serve publicly available static web content (js/css/html) from a Google Cloud Bucket without requiring users to authenticate you need to use a Load Balancer to handle layer 7 TLS traffic.

## Prerequisite

1. Be an admin of `uwit-mci-iam` GCP Project (direct netid@uw.edu GCP IAM applied, no UW Groups).
1. Use a gcloud config that is configured for `uwit-mci-iam`,  save time and [create a profile](new-gcloud-profile.md).

    ```
    gcloud config configurations list
    ```

## Steps

### Create a TLS cert (do once)

We want to serve all static content from `*.cdn.iamprod.s.uw.edu`.  With this kind of setup, each application can have it's own GCP bucket to serve static content without access to write/delete to another apps static content.

1. Create and complete a CSR for `*.cdn.iamprod.s.uw.edu` and save the results to `cdn.key` and `cdn.crt`.

### Create the Load Balancer (do once)

This can be done from the command line or the web console.  Since it's a laborious task from the command line, as long as you have the TLS cert then it's much eaiser to do on the web console.

1. Have at least one GCP Bucket ready to use as a backend in the next step

1. Go to Network Services -> Load Balancing -> Create

1. Use defaults for everything, except, choose `HTTPS` in the "Frontend configuration" and upload the `cdn.key` and `cdn.crt`, also have the intermediate cets available for upload.

### Create A Backend Bucket (do once per app)

This takes multiple steps.  You need to create the bucket, add the backend to the load balancer, create a host and path rule for the load balancer to point to the bucket.  Since each app will need this it is documented in a develper friendly readme at [edit-cdn.md](edit-cdn.md).