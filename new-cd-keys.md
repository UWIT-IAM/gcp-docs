# New SA Key for Deployer
Deployer is the Google Cloud Function used to recieve REST calls from Travis so that Travis can signal that a new container is ready for deployment to k8.

## Prerequisite
1. You are are a member of `u_mciman_tenants_iam_keyadmins`, which grants the permission to generate new keys on restricted service accounts.
1. You have configured a [iamshared gcloud CLI profile](projects-shared.md)
1. You also have configured any respective profiles that you will need (if you need a key for eval then create a `k8eval` profile).

## Creating a Key
2. Switch your `gcloud` profile to the respective profile (k8dev or k8eprod etc)
3. See which accounts exist 

        gcloud iam service-accounts list

1. Using the account that starts with `deploy-0` as the `[account]` value below, create a new key

        gcloud iam service-accounts keys create ./sa.key --iam-account=[account]

1. Now test it...
 