# New K8 Deployment

All deployments to our clusters are automated.  For our guide on automated deployemnts refer to the [gpc-k8](https://github.com/UWIT-IAM/gcp-k8) repo.

You should only be doing manual deployments when experimenting with new architectures in kubernetes.

## Prerequisite

1. You have configured a [k8dev gcloud CLI profile](new-gcloud-profile.md) correctly.

2. You are using the k8dev profile.

    ```
    gcloud config configurations activate k8dev
    ```

## Create Your Deployment

Make a simple deployment with the `gcr.io/google_containers/echoserver:1.3` docker container which once working, you will change the container to be that of your application.

1. When replacing [appname] it best to use your GitHub repo name for your container.

1. Create `./deployment.yml`

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: [appname]
spec:
  replicas: 2
  selector:
    matchLabels:
      app: [appname]
  template:
    metadata:
      labels:
        app: [appname]
    spec:
      containers:
      - name: [appname]
        image: gcr.io/google_containers/echoserver:1.3
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "200m"
          limits:
            memory: "128Mi"
            cpu: "300m"
```

1. Create your deployment `kubectl apply -f ./deployment.yml`

1. Confirm your deployment is running `kubectl get deployments [appname]`

1. By default, containers on GKE are not accessible from the internet and require [modifying an existing load balancer](edit-ingress.md) to see a response from your container.

1. If a load balancer doesn't exist yet for the domain you want to serve your app one then create a [hosted zone](new-hostedzone.md) and [ingress](new-ingress.md).

## Create A Service

The service (NodePort) will provide the networking from the load balancer to your pod.  Without it your pod just sits there with the port exposed with no traffic being sent to it.

1. Create a `./appservice.yml` file with the following contents.

    ```YAML
    apiVersion: v1
    kind: Service
    metadata:
      name: [appname]
      labels:
        app: [appname]
    spec:
      type: NodePort
      ports:
      - port: 80
        targetPort: 8080
        protocol: TCP
        name: http
      selector:
        app: [appname]
    ```

1. Make sure it's running `kubectl get services [appname]`

1. Now you need to [edit an ingress service](edit-ingress.md) so that traffic can find your pod.

## Run Your Deployment On Our Clusters

Put your `service.yml` and `deployment.yml` into the [gcp-k8](https://github.com/UWIT-IAM/gcp-k8) GH repo using the conventions of that repo.