# Shared Redis Cache

There is now a shared redis cache instance accessible for services in our cluster. 
If you would like to use it, read on!

[Redis] is a fast, powerful key-value store.

## Shared Instance Constraints

This is meant to be a caching layer that is performant, with non-critical state. It 
does not save any ledgers of the commands run against it. Nothing is written to disk.
If the instance is restarted, all state will be lost, and any cached entries will 
have to be regenerated. 

If you don't like this, you are welcome to set up a more robust and resilient 
instance yourself.

## Authentication

You will need to obtain credentials in order to secure 
a namespace for yourself or your service. If you are an IAM developer, 
you may be able to do this yourself; see [internal details]. 
Otherwise, contact @tomthorogood, who will be happy to assist you.

## Authorization

Your service user will have access to all commands except dangerous commands. To 
find out which commands are dangerous, run you can use `ACL CAT` (or [see the 
documentation](https://redis.io/commands/acl-cat)).
Additionally, your user will be scoped to keys prefixed with your username (i.e., 
service name). If your service name is `my-app`, all your keys must be prefixed with 
`my-app:`.

This same namespace can be used to obtain the password for the namespace from our 
[secrets].

In the `redis-connection` secret, you will find the hostname and port of the redis
instance.

From the `redis-auth` secret, you can obtain the password for your namespace, like:

```yaml
env:
  - name: REDIS_PASSWORD
    valueFrom:
      secretKeyRef:
        name: redis-auth
        key: my-service-name
```

Similarly, connection information can be loaded via the `redis-connection` secret:

```yaml
env:
  - name: REDIS_HOST
    valueFrom:
      secretKeyRef:
        name: redis-connection
        key: REDIS_HOST
  - name: REDIS_PORT
    valueFrom:
      secretKeyRef:
        name: redis-connection
        key: REDIS_PORT
```

[internal details]: https://github.com/uwit-iam/gcp-k8/tree/master/docs/redis.md
[secrets]: secrets.md
[Redis]: https://redis.io
