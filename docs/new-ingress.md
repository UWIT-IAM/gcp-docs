# New GCloud TLS Ingress Load Balancer

This details the steps required to create a new L7 Ingress load balancer in GCP.  The goal is to use the external IP of this newly created load balancer as the `A` record of your FQDN so traffic can dynamically be routed to your cotnainer within a k8 cluster.

## NOTE

At a minimum, this entire readme must be done once per cluster. It can be done as needed for load balancer redundancy or performance reasons (a single load balancer can serve multiple wildcard TLS certs/traffic)

- [x] `dev` cluster.
- [x] `eval` cluster.
- [x] `prod` cluster.

## Prerequisite

1. You have configured a [iamshared gcloud CLI profile](projects-shared.md)

1. You have a [hosted zone setup](new-hostedzone.md) for `[env].s.uw.edu`.  In this document `[env]` represents `iamdev` for example.

1. You have a [deployment created](new-deployment) called `[appname]` with it's respective service called `[servicename]` (these are often the same name).

    ```
    kubectl get deployment [appname]
    kubectl get services [servicename]
    ```

1. The backing containers of the `[appname]` pods served by `[servicename]` responds with HTTP status code 200 at `/` as a default. Managing health check configuration outside of that route is possible.

1. You also have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md).

1. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

The steps below follow the [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls) documentation.

## TLS Certificate Setup

If you don't have the public and private key for `yourdomain.uw.edu` then you will need to create a new one.

1. Create an InCommon CSR for `*.yourdomain.uw.edu` if you want to serve multiple applications in a cluster with the same certificate. Use the [documentation here](https://wiki.cac.washington.edu/display/infra/Obtain+a+Certificate+from+the+InCommon+CA).
   - For example, if you are serving `app1.yourdomain.uw.edu` then the Common Name (CN) in your CSR could be `*.yourdomain.uw.edu`

2. Save your certs to your filesystem as `yourdomain.uw.edu.crt` and your private key as `yourdomain.uw.edu.key`. This is only temporary and you can delete them when done with this guide.

## Kubernetes Setup

The main documention for this is not yet on GKE docs, but is on [GitHub](https://github.com/kubernetes/ingress-gce) and official [kubernetes.io](https://kubernetes.io/docs/concepts/services-networking/ingress/).  There are numerous ways of setting up ingress, the way it must be done for TLS in GCP is a k8 `service` of type `ingress`.

### 1. Create a K8 TLS Secret

If you run into problems or need help with the encoding use the [secret documentation](https://kubernetes.io/docs/concepts/configuration/secret/).

1. We need to base64 encode your cert and key.  The following command works for Linux (you must remove the line breaks).  Make sure the text for your cert and key end up in a file called `yourdomain.uw.edu.secret.yml`

        base64 -w 0 yourdomain.uw.edu.crt >> encoded.crt
        base64 -w 0 yourdomain.uw.edu.key >> encoded.key

1. Create a `./yourdomain.uw.edu.secret.yml` file with the following.  Make sure to set the value for `tls.crt` and `tls.key` to the value in their respective files you created (single line, no line breaks).

    ```yml
    apiVersion: v1
    data:
      tls.crt: [base64 encoded cert]
      tls.key: [base64 encoded key]
    kind: Secret
    metadata:
      name: yourdomain-tls-secret
      namespace: default
    type: kubernetes.io/tls
    ```
1. Apply the secret `kubectl create -f ./yourdomain.uw.edu.secret.yml`. This can take 5-10 minutes to sync with the GCP Load Blancer as it will refresh what it has if this secret is in use by a K8 Ingress service.

1. You should now see your new secret `kubectl get secrets`

### 2. Create the kubernetes Ingress Service

For every ingress service there is a GCP Load Balancer automatically created for you at $20/month.  Each of these load balancers can serve multiple FQDN's each having their own TLS secret and routing.

A single ingress service You should now have the following before preceeding...

- A k8 secret that has a TLS cert and key.
- A kubernetes deployment of some kind capable of serving http from a pod

1. Determine if you need to create  a new ingress, or, [edit an existing one](edit-ingress.md).

    ```
    kubectl get ingress
    kubectl describe ingress [name]
    ```

1. You can refer to the [kubernetes ingress documenation](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls).  It describes what the Ingress service should be like for a single backend http app that you already have running named `s1` (change that value to match your existing deployment).

1. The documentation also can be used to serve `app1.yourdomain.uw.edu` as well as `app2.yourdomain.uw.edu`.

1. Keep in mind that the secret we created is named `yourdomain-tls-secret` and that will need to be the value for `secretName` in your Ingress yml.

#### Example

1. Create a `./ingress.yml` file with the following...

    ```YAML
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: iamdev-ingress
    spec:
      tls:
      - secretName: iamdev-tls-secret
      backend:
        serviceName: demo
        servicePort: 80
    ```

2. Apply the service, keep in mind GCP takes a good amount of time provisioning and configuring the load balancer when you run this.

    ```
    kubectl apply -f ./ingress.yml
    ```

3. After a good 5-10 minutes, make sure the backend does not say "UNHEALTHY".  This is most likely becuase the port on the container is not set correctly or it doesnt have a health check.

    ```
    kubectl describe ingress [ingress name] |grep backends
    ```

## DNS Setup

We now want to create an `A` record and have it point to the IP of our ingress.

1. Get the IP of the load balancer in use by your ingress.

    ```
    kubectl describe ingress [ingress name] |grep Address
    ```

1. Create an `A` record so networking can be routed to the ingress.  Unfortunately this is a multiline command.

    ```
    gcloud dns record-sets transaction start --zone [zonename]

    gcloud dns record-sets transaction add --zone [zonename] --name [appname].iamdev.s.uw.edu. --ttl 300 --type A "[Load Balancer IP]"

    gcloud dns record-sets transaction execute --zone [zonename]
    ```

1. Wait for the TTL and DNS to propagate, then load `https://[your new A record]` in a browser