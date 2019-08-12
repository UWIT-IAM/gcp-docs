# Creating and editing secrets

## Loading initial secrets

There are two main ways to consume secrets - as files mounted to your pod or as environment variables.
You can mount multiple secrets to a pod, and you may mix methods. In either case, the secrets are stored
via key-value pairs under a secret name.

Consult https://kubernetes.io/docs/concepts/configuration/secret/ for secret-loading. In general, environment
variables are easier to work with. In the case of identity.uw, our main secrets are mounted as files.

## Editing existing secrets

Editing items in a file is a little involved.

```bash
$ kubectl get secrets
NAME                           TYPE                                  DATA   AGE
demo-secret                    Opaque                                1      7s
$ kubectl get secret demo-secret -o yaml
apiVersion: v1
data:
  secret.py: IiIiVGhpcyBpcyB0aGUgc2VjcmV0IGZpbGUhIiIiClBBU1NXT1JEID0gJ2h1bnRlcjInCg==
kind: Secret
...
```

Observe that there is a file named `secret.py`, the value here is a base64-encoded version of the file. Now say you want
to edit the contents of that file. There is a variable `PASSWORD` that needs a new value.

```
$ kubectl get secret demo-secret -o json | jq -r '.data["secret.py"]' | base64 -d
"""This is the secret file!"""
PASSWORD = 'hunter2'
$ kubectl get secret demo-secret -o json | jq -r '.data["secret.py"]' | base64 -d > secret.py.old
$ sed 's/hunter2/hunter3/' < secret.py.old > secret.py
$ base64 -w0 secret.py
IiIiVGhpcyBpcyB0aGUgc2VjcmV0IGZpbGUhIiIiClBBU1NXT1JEID0gJ2h1bnRlcjMnCg==
```

We've changed the password value local to our file. What you want to do now is drop in this new base64 in place of the old one.
You can use `kubectl edit secret demo-secret` with your $EDITOR of choice. Or you can pull down the secret, edit it,
and push it again...

```bash
$ kubectl get secret demo-secret -o yaml > secret.yaml
$ emacs secret.yaml
$ kubectl apply -f - < secret.yaml
secret/demo-secret configured
```

You could even get crafty and do it all from a bash command...

```bash
$ kubectl get secret demo-secret -o json | \
  jq --arg newsecret $(base64 -w0 secret.py) '.data["secret.py"]=$newsecret' | \
  kubectl apply -f -
secret/demo-secret configured
```

As you can see, there are many different ways to shoot yourself in the foot! Pick the one that suits you best.
This is an area that seems ripe for improvement. Possibly on the horizon is the use of Hashicorp Vault. I also
wouldn't be surprised to see a way to edit these values from the GCP console. Time will tell!
