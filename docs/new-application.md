# New Application

## Your New App Architecture

Containerizing an app is half the battle.  Taking something from on prem to the cloud and a container changes the way you develop locally, build, run tests (CI), push aritifacts, pull artifacts (CD), log, monitor, trace and alert.

Please use this guide and expand to it as needed.

### Where To Mount Data/Config

Secrets, config values, variable data, static data...you now need to change how you source this data and inject it into your runtime for local development and a kubernetes runtime.

1. Segment your data by public config values, private config values (secrets), persistant changing data, persistant static data.

2. For each segmentation, choose the appropriate way below to [mount your data](https://cloud.google.com/kubernetes-engine/docs/concepts/volumes).  All of these can and should be used as they meet your needs.

   * [ConfigMap](https://cloud.google.com/kubernetes-engine/docs/concepts/configmap) - Non secret data that can be stored in a Git Hub repo (assume the repo is public).  Assume that your config values are NOT sourced with app code and instead sourced at [gcp-k8](https://github.com/UWIT-IAM/gcp-k8).  Or directly with [environment variables](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/).  Config as code should be brought up to the highest possible level of abstraction is preferred.

   * [Secret](https://cloud.google.com/kubernetes-engine/docs/concepts/secret) - This can be a TLS secret, x509 client cert, api token or any key/value that should NOT be sourced in a Git Hub repo.  GKE secrets are very secure and encrypted at rest.  [TLS secrets](https://github.com/UWIT-IAM/gcp-docs/blob/master/docs/new-ingress.md#kubernetes-setup) are different, but still result in a file mounted to the file system of yoru container.

   * [emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) - An easy to provision (yml) ephermal disk storage.  An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node.

   * [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) - While this uses cloud native storage, its fast, persists forever and may be coordinated with UE ahead of time.  This is a great option for applications that need fast and permenant storage especially for data that is dynamic and must be shared amongst pods (running and future).

   * Others - For static web files you should use a [CDN](new-cdn.md).  You can also use a cloud storage bucket that your app code can access via the `gcloud` or a language specific SDK.  Also managed databases, SQL and NonSQL can be used.

### Mount Data/Config

Now with your data segmented and some storage options selected, you will need to make it so your app can injest the data.  As far as your app is concerned, all of these options result in either env vars or files accessable via the file system.

1. For local development, make your app load this data from env vars (non secret data).  This can be done on your local environment with dot files using something like [dotenv](https://github.com/theskumar/python-dotenv) for python for example.  The idea is, you will have code in your app that uses it's native way of accessing env vars.  When your container runs on Kubernetes, your pod will inject these vars for you when using a this ConfigMap or Env Vars storage option.

2. Get secrets from files.  When doing local development you can source `dummy-secret.pem` to your github repository for your app.  It will not have real values of course but your readme can instruct the developer how to set the necessary values.  During `docker build` locally you can copy these files in.  Your app code can then look for them.  For kubernetes, you mount secrets and persistant volumes so they appear in the same directories/files in the file system for your app.  There are assumptions made by kubernetes and defaults so please understand how these files are created and updated on a per storage option basis.

You should now be able to run your application in a container locally, and, ideally, with the right config stored in [gcp-k8](https://github.com/UWIT-IAM/gcp-k8) it can run in kubernetes.

### Next steps

1. Send all logs/data to stdout and stderr and then [use you logs](get-logs.md).

1. Choose a git branching model that aligns with your [development and deployment patterns](https://docs.google.com/document/d/1ecFyX3HcnE8BGoc8MOvTkXyUozt7ao-0R-T7-iF0v9I/edit).  It does not have to be GitFlow, your master branch represents an artifact that was built in the past.  GitFlow is great for software libraries, but not so much for web applications.

1. Proceed to setting up continous integration so that something can build your container automaticly and push to GCR (below).

1. Enable [ingress](new-ingress.md) from outside of the cluster to your app (if needed).

1. Now with your app running in a cluster, [instrument and monitor](https://docs.google.com/document/d/1LVZ6y0ErvEFooQFUtzycRvujTJj_dnDfZ8RnZrdTT6Q/edit) you application.

## Enable GCR with a Service Account

Google Container Registry AuthZ is done on the GCP Storage Bucket level (access to ALL images in the bucket).  That means all GCP Service Accounts created using this guide will have equal admin/write permissions.

We still want seperate service accounts per app becuase one day google may/should expand their feature set and permit segmenting access by image within GCR.

### Prerequisite

1. Have admin of the `uwit-mci-iam` Google Cloud Project (direct netid@uw.edu GCP IAM applied, no UW Groups).
1. Initialize `gcloud` CLI with the credentials you need, save time and [create a profile](new-gcloud-profile.md).

    ```
    gcloud projects list
    ```

### Setup

1. Using the steps and conventions below you are going to create a [GCP Service Account](https://cloud.google.com/iam/docs/creating-managing-service-accounts) in the `uwit-mci-iam` project.  Please use this naming convention below.

   1. Be consistant with the use of `[appname]` in these scripts, its value should be the name of the app git repo if possible.
   1. Create the Service Account

        ```
        gcloud iam service-accounts create [appname]-gcr-ci --display-name "[appname] docker registry admin"
        ```

   1. Grant storage admin permissions to this SA for our GCR bucket

        ```
        gsutil iam ch serviceAccount:[appname]-gcr-ci@uwit-mci-iam.iam.gserviceaccount.com:admin gs://artifacts.uwit-mci-iam.appspot.com
        ```

   1. (Optional) Verify the permissions set for your new account, you should see a role of `roles/storage.admin` for your new account.

       ```
       gsutil iam get gs://artifacts.uwit-mci-iam.appspot.com
       ```

1. Using gcloud, download a long lived credential as a [JSON Key File](https://cloud.google.com/container-registry/docs/advanced-authentication#json_key_file) for this new service account (this will be used by Travis).

    1. Create the key.

        ```
        gcloud iam service-accounts keys create ./key.json --iam-account [appname]-gcr-ci@uwit-mci-iam.iam.gserviceaccount.com
        ```

    1. (Optional) test the key (you should see `Login Succeeded`).

        ```
        cat key.json | docker login -u _json_key --password-stdin https://gcr.io
        ```

    1. We need this key for the next section.  Base64 encode the key, copy the output.

        ```
        base64 -w 0 key.json
        ```

## Setup CI/Travis

### Prerequisite

1. You have a working dockerfile in a github repo

### Setup

1. Create a new Environment Variable in Travis named `GCR_ADMIN_TOKEN` with a value of the base64 encoded copied output from the previous step.

1. Create a new Environment Variable in Travis named `IMAGE_NAME` and set its value to what you are using for `[appname]`.

1. In your app repo create a `.travis.yml` with the following ([read their docs](https://docs.travis-ci.com/) for how to customize).

    ```YAML
    language: python
    install:
      - echo $GCR_ADMIN_TOKEN | base64 -d | docker login -u _json_key --password-stdin https://gcr.io
    env:
      - DOCKER_ROOT=gcr.io/uwit-mci-iam/
    before_script:
      - docker version
    script:
      - docker build -t ${IMAGE_NAME} .
      - docker tag ${IMAGE_NAME} ${DOCKER_ROOT}${IMAGE_NAME}:V1.0.0
      - docker push ${DOCKER_ROOT}${IMAGE_NAME}:V1.0.0
    done
    ```

1. Commit and push to your git repo with this new travis file, make sure Travis is set to automatically build.

1. Verify your build works, get help from the team if it doesnt, update these docs as needed.

## Next Steps

1. Your docker container must serve a `200` by default on `/` as that is required for GKE Ingress health checks.  Otherwise, serve a `200` on another route and specify the readiness and health checks in the same way as the `identityuw` deployment.

1. Everything must be working for your to proceed.  Deployment to Kubernetes is fully automated, even for the development cluster. Please make sure the steps above all work.

1. Your app is now ready for Continous Integration.  But, there is much to consider when architecting automation to be in sync or not with your Git branching model and release model.  Please [read this document](https://docs.google.com/document/d/1ecFyX3HcnE8BGoc8MOvTkXyUozt7ao-0R-T7-iF0v9I/edit) and discuss with a team member.

1. Before attempting to get your app running a K8 cluster it's best to think and discuss it's architecture first.  Setting aside all the CI automation concerns in the step above, what "state" if any does your app have...a database, a cache, static files?  Remember that your container must be designed to be stateless (ideally, statefull things are possible).  Discuss with the team and come up with a plan.

1. Now you are finally ready to deploy to K8, have a look at the guides on the [main readme](README.md) for more on that.