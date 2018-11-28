# Edit GCloud TLS Ingress Load Balancer
This assumes you already have created an ingress.  You would edit an ingress only when you need to add a new FQDN.

## Prerequisite
1. You have [configured an ingress](new-ingress.md) 
    
1. The backing containers of the `[appname]` pods served by `[servicename]` responds with HTTP status code 200 at `/` as a default. Managing health check configuration outside of that route is possible.

1. You also have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md).
1. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## DNS Setup
1. Get information about an existing hosted zone, use an `[ip]` in the next step and `[zonename]` of an existing zone. 

    ```
    gcloud dns record-sets list --zone=iamdev
    ```

1. Create an `A` record so networking can be routed to the ingress.  Unfortunately this is a multiline command.

    ```
    gcloud dns record-sets transaction start --zone [zonename]

    gcloud dns record-sets transaction add --zone [zonename] --name [fqdn]. --ttl 300 --type A "[ip]"

    gcloud dns record-sets transaction execute --zone [zonename]
    ```

## Kubernetes Ingress Setup
1. Get the ingress that you want to change...
1. Make sure you have a service running and ideally a deployment using that service...
2. Patch the ingress with the new yaml...
