# New Load Balancer for CDN, Buckets and Static Content

In order to serve publicly available static web content (js/css/html) from a Google Cloud Bucket without requiring users to authenticate you need to use a Load Balancer to handle layer 7 TLS traffic.

## Prerequisite

1. Be an admin of `uwit-mci-iam` GCP Project (direct netid@uw.edu GCP IAM applied, no UW Groups).
1. Use a gcloud config that is configured for `uwit-mci-iam`,  save time and [create a profile](new-gcloud-profile.md).

    ```
    gcloud config configurations list
    ```

## Steps

### Create a TLS cert (do whenever it expires)

We want to serve all static content from `*.cdn.iamprod.s.uw.edu`.  With this kind of setup, each application can have it's own GCP bucket to serve static content without access to write/delete to another apps static content.

1. Generate a new Certificate Signing Request (CSR) for the CDN: `openssl req -new -newkey rsa:2048 -nodes -keyout identity-cdn.key -out identity-cdn.csr`, and use the following values:
> Country Code: US

> State: WA

> Locality: Seattle

> Organization: University of Wahington

> Organizational Unit: UW-IT IAM

> Common Name: \*.cdn.iamprod.s.uw.edu

> email address: yours!

2. Use the UW [Certificate Service] to create a new **InCommon** certificate:

> CSR (PEM): Paste contents from identity-cdn.csr (above)

> AltNames: Leave blank

> Type: SSL

> Server: Other

> Number of Servers: 1

> Lifetime: 1 year

3. Wait for an email telling you that your certificate has been issued (or refresh the [Certificate Service] page until your new cert shows as "issued")
4. In a text editor, paste the PEM from the certificate (find this by going to your new certificate in the [Certificate Service] and then clicking on "Get PEM")
5. Paste the PEM into your text editor
6. Then, copy the contents of the "InCommon intermediate certificates for sha-2 certificates signed after October 5, 2014." and paste that into the text editor _after_ what you've already pasted. 
7. Open the [Google Load Balancers Certificate Management] page (make sure you are in the uwit-mci-iam project!) -- If that link doesn't work, click on the button that says "Go to the certificates page" [here](https://cloud.google.com/load-balancing/docs/ssl-certificates/google-managed-certs) -- There is no other way to get there through the UI.
8. Click on "Create SSL certificate"

> Name: identity-cdn-{upcoming year}-incommon

> Create mode: (\*) upload my certificate

> Certificate: Paste the contents of the text editor you have open that includes your certificate PEM _and_ the intermediate certificates!

> Private Key: Click "Upload" then select the `identity-cdn.key` you generated in the first step above.

9. Click "Create"


### Create the Load Balancer (do once)

This can be done from the command line or the web console.  Since it's a laborious task from the command line, as long as you have the TLS cert then it's much eaiser to do on the web console.

1. Have at least one GCP Bucket ready to use as a backend in the next step

1. Go to Network Services -> Load Balancing -> Create

1. Use defaults for everything, except, choose `HTTPS` in the "Frontend configuration" and upload the `cdn.key` and `cdn.crt`, also have the intermediate cets available for upload.

### Create A Backend Bucket (do once per app)

This takes multiple steps.  You need to create the bucket, add the backend to the load balancer, create a host and path rule for the load balancer to point to the bucket.  Since each app will need this it is documented in a develper friendly readme at [edit-cdn.md](edit-cdn.md).

[Certificate Service]: https://iam-tools.u.washington.edu/cs/
[Google Load Balancers Certificate Management]: https://console.cloud.google.com/loadbalancing/advanced/sslCertificates/list?_ga=2.219913497.145386408.1608258050-2027690782.1593017906

# Ongoing Maintenance

**NOTE**: In future years, it _should_ no longer be necessary to also update the 
private key, so before you follow these instructions in December 2021, check and see 
if it's actually required!

Each year (or until we can use a certificate manager for the CDN) the certificate must be updated.

1. Follow all the steps [above](#create-a-tls-cert-do-whenever-it-expires)
1. [Edit the CDN](https://console.cloud.google.com/net-services/loadbalancing/edit/http/cdn-iamprod?project=uwit-mci-iam&organizationId=657476903663)
1. Click on "Frontend Configuration"
1. Click the pencil icon
1. In the dropdown box under "certificate", select your new certificate.
1. Click "Update"
