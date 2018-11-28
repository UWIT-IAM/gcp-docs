# New Application
This is meant to be done by admins of the `uwit-mci-iam` Google Cloud Project.  It has very restricted access becuase it is the project that includes "shared" or "common" things across all our projects like Cloud DNS, the Docker image registry and more.

## Prerequisite
1. Have admin of the `uwit-mci-iam` Google Cloud Project (direct netid@uw.edu GCP IAM applied, no UW Groups).
1. Make sure you have initialized the `gcloud` CLI with the credentials you need as well as set to the correct projec. Maybe [create a profile](new-gcloud-profile.md) for this?

    ```
    gcloud projects list
    ```
    
2. You have a working dockerfile in a github repo
3. You have some minimal setup for that repo at https://travis-ci.org/

## Create A GCR Service Account    
Google Container Registry AuthZ is done on the GCP Storage Bucket level (access to ALL images in the bucket).  That means all Service Accounts created using this guide will have equal admin/write permissions.

Kubernetes clusters however, already configured, are limited to read only permissions.

We still want seperate service accounts per app becuase one day google may/should expand their feature set and permit segmenting access by image within GCR.


1. Using the steps and conventions below you are going to create a [GCP Service Account](https://cloud.google.com/iam/docs/creating-managing-service-accounts) in the `uwit-mci-iam` project.  Please use this naming convention below.
   1. Be consistant with the use of `[appname]` in these scripts, its value should be the name of your git repo if possible
   1. Create the Service Account

        ```
        gcloud iam service-accounts create [appname]-gcr-ci --display-name "[appname] docker registry admin"
        ```

   2. Grant storage admin permissions to this SA for our GCR bucket

        ```
        gsutil iam ch serviceAccount:[appname]-gcr-ci@uwit-mci-iam.iam.gserviceaccount.com:admin gs://artifacts.uwit-mci-iam.appspot.com
        ```
        
   3. (Optional) Verify the permissions set for your new account, you should see a role of `roles/storage.admin` for your new account.
   
       ```
       gsutil iam get gs://artifacts.uwit-mci-iam.appspot.com 
       ```

1. Using gcloud, download a long lived credential as a [JSON Key File](https://cloud.google.com/container-registry/docs/advanced-authentication#json_key_file) for this new service account (this will be used by Travis). 
    1. Create the key in your pwd.
    
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
1. Create a new Environment Variable in Travis named `GCR_ADMIN_TOKEN` with a value of the base64 encoded copied output from the previous step.
2. Create a new Environment Variable in Travis named `IMAGE_NAME` and set its value to what you are using for `[appname]`.
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
1. Everything must be working for your to proceed, manually deployment of apps in eval and prod is not permitted and is done via a CI to CD integration.  Please make sure the steps above all work.
1. Your app is now ready for Continous Integration.  But, there is much to consider when architecting automation to be in sync or not with your Git branching model and release model.  Please [read this document](https://docs.google.com/document/d/1ecFyX3HcnE8BGoc8MOvTkXyUozt7ao-0R-T7-iF0v9I/edit) and discuss with a team member.
2. Before attempting to get your app running a K8 cluster it's best to think and discuss it's architecture first.  Setting aside all the CI automation concerns in the step above, what "state" if any does your app have...a database, a cache, static files?  Remember that your container must be designed to be stateless (ideally, statefull things are possible).  Discuss with the team and come up with a plan.
3. Now you are finally ready to deploy to K8, have a look at the guides on the [main readme](README.md) for more on that. 