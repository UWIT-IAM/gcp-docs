# New Cloud DNS Hosted Zone

This details the steps required to create a new hosted zone.  An example would be you wanting to serve traffic to an A record of `[appname].iamdev.s.uw.edu`.  A hosted zone is needed for `iamdev.s.uw.edu` before you can route traffic into GCP services.

## Prerequisite
1. You have setup access to the [IAM shared GCP project](projects-shared.md)
2. You are using the iamshared profile.

    ```
    gcloud config configurations activate iamshared
    ```

1. Be the DNS admin of a [UW subdomain](https://itconnect.uw.edu/connect/uw-networks/network-addresses/requesting-a-new-subdomain/uw-subdomain/) `yourdomain.uw.edu`.  This readme uses that domain as an example and will servce `*.yourdomain.uw.edu` if you like.

## Setup Google Cloud DNS
This is only needed once per `yourdomain.uw.edu` or `*.yourdomain.uw.edu` that you want to network.


1. In these instructions use...
   1. `[zonename]` = the `iamdev` of `iamdev.s.uw.edu` for example
   2. `[dnsname]` = all of `iamdev.s.uw.edu` for example

1. Create the managed zone

    ```
    cloud dns managed-zones create [zonename] --dns-name="[dnsname]" --description="Describe the purpose for this zone"

    gcloud dns managed-zones list
    ```

1. Send the `NS` and `SOA` recrods in an email to `help@uw.edu` and ask them to update your records for `[dnsname]` to include the new  GCloud `NS` and `SOA` records.

   ```
    gcloud dns record-sets list --zone="[zonename]"
   ```