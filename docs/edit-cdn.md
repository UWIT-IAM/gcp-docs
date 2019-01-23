# New Bucket for Serving Static Content

It's ideal to serve static content like `js/css/html` from a CDN vs. within a container using app layer code.  This readme will guide you towards creating a GCP bucket with public access along with editing our CDN GCP Load Balancer to route traffic from `yourapp.cdn.iamprod.s.uw.edu` to your bucket.

## Prerequisite

1. Be an admin of `uwit-mci-iam` GCP Project (direct netid@uw.edu GCP IAM applied, no UW Groups).

1. Use a gcloud config that is configured for `uwit-mci-iam`,  save time and [create a profile](new-gcloud-profile.md).

    ```
    gcloud config configurations list
    ```

1. Have an application that you are already serving and will use here for the `[APPNAME]` variable below.  The value of `[APPNAME]` must be DNS friendly as it will result in `APPNAME.cdn.iamprod.s.uw.edu` being used to serve your static content.

## Steps

This bucket will be 100% public and multi-regional, do not try to shim in content and set ACL's at the folder or file level, there are better ways of doing that outside of this bucket you are creating.

1. Create the bucket and make it public with `AllUsers` ACL.

    ```
    gsutil mb gs://uwit-iam-[APPNAME]-static
    gsutil iam ch allUsers:objectViewer gs://uwit-iam-[APPNAME]-static
    ```

1. Grant your CI Service Account access to upload to this bucket.  You created this account during the [new-application](new-application.md) process.

    ```
    gsutil iam ch serviceAccount:[APPNAME]-gcr-ci@uwit-mci-iam.iam.gserviceaccount.com:admin gs://uwit-iam-[APPNAME]-static
    ```

1. Add your new bucket as a [Backend to Cloud CDN](https://cloud.google.com/load-balancing/docs/backend-bucket)

    ```
    gcloud compute backend-buckets create uwit-iam-[APPNAME]-static --gcs-bucket-name uwit-iam-[APPNAME]-static --enable-cdn
    ```

1. Add a forwarding rule to our existing Load Balancer to send traffic to your new Backend.

    ```
    gcloud compute url-maps add-path-matcher cdn-iamprod \
      --path-matcher-name uwit-iam-[APPNAME]-static \
      --default-backend-bucket=uwit-iam-[APPNAME]-static \
      --new-hosts [APPNAME].cdn.iamprod.s.uw.edu
    ```

1. Create an `A` record so that traffic resolves to the forwarding URL.  The IP used here is that of the `cdn-iamprod` Load Balancer.

    ```
    gcloud dns record-sets transaction start --zone iamprod

    gcloud dns record-sets transaction add --zone iamprod --name [APPNAME].cdn.iamprod.s.uw.edu. --ttl 300 --type A 35.244.145.16

    gcloud dns record-sets transaction execute --zone iamprod
    ```