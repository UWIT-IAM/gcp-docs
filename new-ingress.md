# TO DO - write this just for the ingress part



This details the steps required to get TLS working in GKE with a new Ingress service or updating an existing one.  

## 1. Pre-Requisites

1. Have access to a GCP project.
1. Created a GKE cluster in that project, [the quickstart](https://cloud.google.com/kubernetes-engine/docs/quickstart) gets you up in running in less than 5 minutes.
1. Configured a [GKE shell](https://cloud.google.com/kubernetes-engine/docs/quickstart) for the project/cluster and are able to run `kubectl` commands.
1. A [kubernetes deployment](https://cloud.google.com/kubernetes-engine/docs/how-to/stateless-apps) capabale of serving http traffic.
1. Be the DNS admin of a [UW subdomain](https://itconnect.uw.edu/connect/uw-networks/network-addresses/requesting-a-new-subdomain/uw-subdomain/) `yourdomain.uw.edu`.  This readme uses that domain as an example to ultimately serve `app.yourdomain.uw.edu` with TLS.  This will also work for serve `yourdomain.uw.edu` as well with a matching `A` record.

## 2. DNS
You can later create an `A` record whereever you currently manage your domain, or, use GCP Cloud DNS.  Using Cloud DNS can give you better GR as GCP's SLA is better than UW-IT.

### Do This Once Per Subdomain
1. In your GCP project, navigate to Network Services - Cloud DNS
1. Create a new zone for `yourdomain.uw.edu`
1. Click on the list of your zones and select `yourdomain.uw.edu` once it's created
1. Copy and paste the `NS` and `SOA` to an email to `help@uw.edu` and ask them to update your records for `yourdomain.uw.edu` to include this new `NS` and `SOA`.

Since we dont yet have an external IP for your TLS Ingress service (GKE/Google automaticly creates this for us) we can't yet create `A` records.  We will revisit this shortly including using an existing Ingress service.

## 3. Create the TLS kubernetes Secret
Kubernetes needs a way for the Ingress service to find and use a TLS cert.  The public and private key are stored as kubernetes secrets.  These steps follow the [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls) documentation.

### Create a new cert
If you don't have the public and private key for `yourdomain.uw.edu` then you will need to create a new one.

1. Create an InCommon CSR for `*.yourdomain.uw.edu` if you want to serve multiple applications in a cluster with the same certificate, otherwise, don't make your cert be wildcard. Use the [documentation here](https://wiki.cac.washington.edu/display/infra/Obtain+a+Certificate+from+the+InCommon+CA). 
1. For example, if you are serving `app1.yourdomain.uw.edu` then the Common Name (CN) in your CSR could be `*.yourdomain.uw.edu`
1. Once the CSR is approved, the rest of this readme will use `yourdomain.uw.edu.key` and `yourdomain.uw.edu.crt` for the key and public cert respectively.

### Use an existing cert
If you have the public and private key for `yourdomain.uw.edu` use the following steps to create a TLS kubernetes secret.  If you run into problems or need help with the encoding use the [secret documentation](https://kubernetes.io/docs/concepts/configuration/secret/).


1. We need to base64 encode your cert and key.  The following command works for Linux (you must remove the line breaks).  Make sure the text for your cert and key end up in a file called `yourdomain.uw.edu.secret.yml`

        base64 -w 0 yourdomain.uw.edu.crt >> iamdev.s.uw.edu.secret.yml
        base64 -w 0 yourdomain.uw.edu.key >> iamdev.s.uw.edu.secret.yml

1. Edit the `yourdomain.uw.edu.secret.yml` file to be the following.  Make sure to set the value for `tls.crt` and `tls.key` correctly (single line, no line breaks)

    ```yml
    apiVersion: v1
    data:
      tls.crt: base64 encoded cert
      tls.key: base64 encoded key
    kind: Secret
    metadata:
      name: yourdomain-tls-secret
      namespace: default
    type: Opaque
    ```
1. Apply the secret `kubectl create -f ./yourdomain.uw.edu.secret.yml`

## 4. Create the kubernetes Ingress Service
You should now have the following before preceeding...

- The ability to create/edit `A` records for `yourdomain.uw.edu`
- A TLS cert and key as a kubernetes secret
- A kubernetes deployment of some kind capable of serving http from a pod

1. At this point you must follow the [kubernetes ingress documenation](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls).  It describes what the Ingress service should be like for a single backend http app that you already have running named `s1` (change that value to match your existing deployment).  The documentation also can be used to serve `app1.yourdomain.uw.edu` as well as `app2.yourdomain.uw.edu`.  Keep in mind that the secret we created is named `yourdomain-tls-secret` and that will need to be the value for `secretName` in your Ingress yml.
1. With your Ingress service created, you now need to go to VPC Networks - External IP Addresses.
1. Find the IP in the list that was created for you, switch it to `Static` if it isnt already.
1. Using the same IP address, go to Cloud DNS (or whever you manage DNS) and create an `A` record pointing `yourdomain.uw.edu` to this new IP.
1. Wait for the TTL and load https://yourdomain.uw.edu in a browser

