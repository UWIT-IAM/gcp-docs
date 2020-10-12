# New GCloud TLS Ingress Load Balancer

This details the steps required to create a new L7 Ingress load balancer in GCP.  These will have a static IP that can be used for DNS so traffic can dynamically be routed to your cotnainer within a k8 cluster.

These load balancers can not do client certificate authentication, that must be done with an different load balancer.

This guide focuses on `host` based routing.  You can also do `name` based routing which enables `http://host.uw.edu/app1` to route to container `app1` for example.  Named based routing is simplier to automate and manage.

## NOTE

At a minimum, this entire readme must be done once per cluster. It can be done as needed for load balancer redundancy
or performance reasons. A single load balancer can serve multiple wildcard TLS certs/traffic. See
[Using multiple SSL certificates in HTTP(s) load balancing with Ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-multi-ssl) for further information.

- [x] `dev` cluster.
- [x] `eval` cluster.
- [x] `prod` cluster.

## Prerequisite

1. You have a static IP registered by Unix Engineering as a named object, something like `gke-ingress-1`.

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


## Kubernetes Setup

The main documention for this [is here](https://wiki.cac.washington.edu/display/MCI/Kubernetes+Ingress). We use the nginx ingress solution provided by UW-IT UE. This solution also manages our TLS certificates using [LetsEncrypt](https://letsencrypt.org/), which simplifies our operations.


### Example

1. Create a `./ingress.yml` file with the following.  This assumes that there is a service `demo` already running in the 
cluster that has something behind it capable of serving traffic. 

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: demo
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt  # This certificate will be created and managed by kubernetes
spec:
  tls:
  # For each TLS certificate, you need one entry like this one.  
    - hosts:
        - demo.iamdev.s.uw.edu  # You can list multiple hostnames per TLS certificate if you choose to
      secretName: demo-cert-demo.iamdev.s.uw.edu  # This secret will be created and managed by kubernetes, see secrets.yml for more info.

  rules:
    - host: demo.iamdev.s.uw.edu
      http:
        paths:
          - path: /
            backend:
              serviceName: demo
              servicePort: 80
```

#### Automated Deployment

1. Put this file in the respective environment folder at https://github.com/UWIT-IAM/gcp-k8
1. Follow the instructions at that repo for more details.

#### Manual Deployment

**Note:** Manual deployments are not recommended, because they will quickly be overwritten by Flux-Weave, which will 
detect that the deployed configuration is different from that stored in the git repository, and re-apply what is
committed.

```
kubectl apply -f ./ingress.yml
```

Changes should take place almost immediately.


## DNS Setup

We now want to create an `A` record and have it point to the IP of our ingress.

1. Get the IP of the load balancer in use by your ingress.

    ```
    kubectl describe ingress [ingress name] |grep Address
    ```

1. Create a `CNAME` record so networking can be routed to the ingress.

    ```
    gcloud dns record-sets transaction start --zone [zonename]

    gcloud dns record-sets transaction add --zone [zonename] --name [appname].iamdev.s.uw.edu. --ttl 300 --type CNAME "[dns for a service already with an A record]"

    gcloud dns record-sets transaction execute --zone [zonename]
    ```

1. Wait for the TTL and DNS to propagate, then load `https://[your new A record]` in a browser

