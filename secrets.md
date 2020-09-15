**Note: This information is visible to the public. 
Do not include sensitive information about secrets management here.**

# Secrets Overview

A _secret_ is anything required at runtime that we don't want to commit into a code repository. Typical
use cases for secrets include:

* Credentials and tokens for API dependencies
* Certificates
* Encryption Keys

At deploy time, our Kubernetes cluster "deploys" secrets in various ways so that they can be accessed by our 
applications. 

The secrets' _data_ are stored in an internal instance of the Hashicorp Vault, managed by UW-IT Unix Engineering.

All Kubernetes cluster configuration is stored in the access-controlled gcp-k8 repository.

## Terminology

**Kubernetes Secret** is a Kubernetes resource of type "Secret." The resource itself contains metadata about the secret,
and its base64-encoded content.

**External Secret** is a Kubernetes resource of type "ExternalSecret." External secrets are the interface
between Kubernetes and the Hashicorp Vault. 

The **Vault** is the secret data and version management service vended by Hashicorp.


# Secret Naming

We have a naming scheme for secrets that allows for deterministic names _and_ Vault paths.

The format of the naming scheme looks like this:

`{product}-[{short-qualifier}-]{functional-description}`

Within the vault, the above scheme translates to the path:

`{stage}/{product}/[{qualifier}/]{functional-description}`

where:

**Stage** refers to whether the secret is targeting the `dev`, `eval`, or `prod` stage of the application.

**Product** refers either to the product consuming the secret, or the product that the secret grants access to.

**Qualifier** is an optional and typically discouraged layer that can be used to group several related
secrets together. It is discouraged to avoid unnecessary nested. Currently we only a qualifier for certificates,
as we often have more than one certificate-adjacent secret.

**Short Qualifier** is a truncated or otherwise shortened version of the above qualifier. ("certificate" becomes "cert"),
and is used for quality of life when interacting with `kubectl`, to avoid overlong names in terminal tables.

**Functional Description** is a short (1–3 word) description of what the secret does or how it behaves. 

So, a secret named `identity-uw-cert-identity.uw.edu` would describe a certificate, used by the `identity-uw` workload,
for its `identity.uw.edu` endpoint. 

Within Vault, the path for the actual secret data would translate to: `{stage}/identity-uw/certificates/identity.uw.edu`.


# Secrets Operations


## Rotating Secrets

If you need to rotate the values of your secrets, you can do that completely within the Hashicorp Vault UI. To do so,
you must belong to the correct permissions group managed by the UW-IT UE team. 

For more information, please refer to the UW-IT UE documents on the [Vault](https://wiki.cac.washington.edu/display/MCI/Mosler+Secrets+Vault). 
This documentation is not publicly accessible.

Changing the value of a secret in Vault is done by creating a new _version_ of the secret. The mere act of 
creating this new version will make use of the Kubernetes External Secrets configuration to ensure the new values
are deployed in near real-time. However, depending on how and when your application is accessing those secrets,
your application may not start using the new version until is is restarted. 

## Creating Secrets

Creating a new secret involves the following steps:

1. Creating the actual secret in the Vault
1. Creating the ExternalConfiguration item for the Kubernetes cluster.
1. Applying the ExternalConfiguration item to the Kubernetes cluster.

The UW-IT IAM gcp-k8 package contains a utility to do this for you, with additional checks and helpful links to 
streamline the process. The utility automatically generates the fully qualified name and Vault path for you and is 
run like so:

```
python util/secret_manager.py \
    create-external-secret \
    --cluster foo-bar-0001-cluster \
    --stage dev \
    --workload my-site \
    --qualifier certificate \
    --secret-name www.mysite.com \

# This would ultimately create a secret called `my-site-cert-www.mysite.com` 
# with the Vault path `dev/my-site/certificate/www.mysite.com`.
```

The above command will:

- Prompt you to create the secret value in the Vault, and tell you what to paste into the Vault form.
- Create the ExternalSecret entry in `gcp-k8/dev/my-site/secrets.yml`
- Apply the configuration to your cluster
- Verify that the secret was synced from Vault

You can also provide a `--dry-run` flag to roll back all changes at the end of the operation, and a `--no-apply` flag
to create the entry only, but not apply it to your cluster.

## Viewing Secrets

There are several ways to view secrets. The easiest is by using the Vault UI.

You can also use `kubectl` to view secrets: `kubectl get secret my-site-cert-www.mysite.com -o json` will show you the 
metadata and base64-encrypted data of the secret created in the above example.

You can use `kubectl` to view `ExternalSecrets` also: `kubectl get externalsecret my-site-cert-www.mysite.com -o json`,
for example. This is useful if you suspect a syncing problem between Vault and Kubernetes; the external secret will 
let you know if there is an error.

To view the secret values that aare stored in the kubernetes secret, you can use a piped command like:

`kubectl get secret my-site-cert-www.mysite.com -o json | jq -r '.data["tls.key"]' | base64 -d`, which would extract the
cert key from your certificate json data, then base64 decode it for you.

Lastly, you can simply use use the secret manager utility:

```
python util/secret_manager.py  dump-secret --secret name my-site-cert-www.mysite.com --data-only
```

This would spit out the full value of the secret, which should exactly match what is stored in Vault.


## Deleting Secrets

1. Delete the `ExternalSecret` configuration item from the Kubernetes config
1. Delete the `ExternalSecret` metadata from the Kubernetes cluster
1. Delete the secret in Vault.


Step 1 involves literally just deleting the blob in the relevant `secrets.yml` file within the gcp-k8 package. This has 
to be done first, or else Flux will just re-populate everything else from this configuration.

For step 2, you can use `kubectl`: `kubectl delete externalsecret my-site-cert-www.mysite.com`. Note that we're deleting
the _external_ secret here; if you just delete the `secret`, Flux will re-create it from the `ExternalSecret` configuration.

Last, and only once you've validated that your application can still run without this secret, you can delete the data
itself from Vault using the UI.

There is currently no scripted way to perform this operation.
