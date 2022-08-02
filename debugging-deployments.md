## Debugging Stuck Deployments

Recommended CLI tools:

- `kubectl`
- `flux`
- `gcloud`

Refer to [gcloud-kubectl-cli.md](gcloud-kubectl-cli.md) for more information on 
configuring and using these tools.


**I pushed a change I expected to deploy, but nothing happened; now what?**

First, you need to find out where in the process your deployment got stuck.
Start with [Was your image tagged?](#was-your-image-tagged) and follow the prompts 
from there.


### Was your image tagged?

- **Was your docker image created?** Visit https://gcr.io/uwit-mci-iam and look for your
  app's docker image. (e.g., 'identity-uw' or 'husky-directory') - Click on it. Do 
  you see a tag with `deploy-<stage>.YYYY.MM.DD.hh.mm.ss` whose timestamp matches 
  what you would expect? 
    - [Yes, I see the tag](#did-flux-see-your-image)
    - [No, there is no tag](#manually-tag-and-push)


### Manually tag and push

If you can't find your deployment tag, something didn't get pushed.
This usually means that either 
(a) your image failed to build, or 
(b) who/whatever did the push did not have sufficient permissions.

Check the output of whatever ran the deployment (i.e., Github Actions) to see 
whether (a) or (b) are the most likely cause.

If the image failed to build, use the output to resolve the issue; this could be
cause by anything from testing to a typos in the dockerfile. 

If it looks like a permissions issue, verify that the entity has the
permissions required to access the repository 
([gcloud docs](https://cloud.google.com/container-registry/docs/access-control)).


To unblock yourself quickly, you can manually tag and push.

If there is no tag, and you want to sidestep the 
automation to deploy in an emergency, you can 
tag and push the deployment image yourself.

Pull the version you want to deploy, then tag it:

```
docker pull gcr.io/uwit-mci-iam/app-name:9.9.9
docker tag gcr.io/uwit-mci-iam/app-name:deploy-dev.2022.08.02.12.00.v9.9.9 
docker push gcr.io/uwit-mci-iam/app-name:deploy-dev.2022.08.02.12.00.v9.9.9
```

### Did flux see your image?

Visit the [gcp-k8] repository to make sure that flux saw your new image tag. 
Check `<stage>/<app-name>/helm-release.yaml` and look for the the `image:` block;
the `tag` field should have a timestamp that matches your deployment tag 
(e.g., `deploy-dev.2022.08.02.12.00.v9.9.9`). If the timestamp is further in the past,
then the issue is something with your image.

Did flux see your image?

- [Yes, flux saw my image](#check-the-chart-reconciliation)
- No, flux did not see my image, I should keep reading:


Since you have already [made sure your image was tagged](#was-your-image-tagged), 
ask yourself:

1. [Is flux looking for the right tag?](#is-flux-looking-for-the-right-tag)
2. [Is the image policy reference correct?](#is-the-image-policy-reference-correct)

#### Is flux looking for the right tag?

In the [gcp-k8] repository, check that the `<stage>/<app-name>/automation.yaml` 
meets the following conditions:

1. The `ImagePolicy` document is present and:
   1. References the `ImageRepository` defined in the same file.
   2. Uses a regex `pattern` that matches the tag you expect to have been deployed
   3. The `namespace` for the policy is `default`.
2. The `ImageRepository` document is present and
   1. References the repository present in the `helm-release.yaml` `.image.repository` field.
   2. Links to a valid repository (paste the `gcr.io/...` link into your browser).
3. Use a regex pattern that matches the pattern supplied in the `ImagePolicy` document.
   (Use any regex validation tool to make sure these two things agree.)
4. Reference the `<app-name>-gcr` `ImageRepository`

If this all check out, move onto: [Is the image policy reference correct?](#is-the-image-policy-reference-correct).


#### Is the image policy reference correct?

The image policy reference is included as a comment in the `helm-release.yaml`, which
makes it particularly susceptible to typos as files are edited. 

In the `helm-release.yaml`, look for the `image:` block; nested inside you will see 
the `tag:` field. After whatever value the field holds, there should be a comment 
that follows the pattern: `# {"$imagepolicy": "default:<app-name>-policy:tag"}`.

The key should *always* be `"$imagepolicy"` -- no caps!

1. Verify that both the key and value are in quotes
2. Verify the entire contents of the comment are valid JSON
3. Verify the `<app-name>-policy` matches the name of the policy from 
   [the last step](#is-flux-looking-for-the-right-tag).

If all of this seems OK, [ask for help!]

### Check the chart reconciliation

After flux notices your new tag and updates [gcp-k8], which you already verified 
[above](#did-flux-see-your-image), another workflow will be 
triggered that will take the values from your `helm-release.yaml` and use the [helm 
chart] to create your kubernetes resources.

To check whether the chart was successfully reconciled, you will need to use the 
[kubectl cli]. 

```
kubectl get helmrelease <app-name>
```

- If the release shows as "reconciled,"
  re-run the command with `-o json` and make sure the timestamps are recent. (i.e., 
  since you started your deployment). If not, 
  [validate your release is not suspended](#resume-a-suspended-release).
- If the release shows "Upgrade retries exceeded", re-run the above command with `-o 
  json` to get more details about the failure. Do your best to dig in, and resolve 
  the issue; if you get stuck, [ask for help!]
- If the release timed out waiting for the release to be ready, you 
  [should check the container logs](#check-container-logs). 
  This usually indicates a problem with the service readiness state.

#### Resume a suspended release

Sometimes the best way to resolve a flux issue is to turn it off and then 
turn it back on again:

```
flux suspend helmrelease <app-name> -n default
flux resume helmrelease <app-name> -n default
```

If you've already tried this once, just [ask for help!] instead.

This call will block on reconciliation; use the results from this call to 
return to the [last step](#check-the-chart-reconciliation) and make a decision
based on the reconciliation results. 

### Check container logs

To check the container logs, you first have to know which pods are new (failing to 
be ready), and which are old (still serving your old application version).

To do so, use the `kubectl` CLI:

```
kubectl get pods
```

This will show all pods inyour cluster, look for the ones matching your application 
name. (If you don't see any, [ask for help!]) One should be older than the others; 
this is the one whose logs you want to check:

```
gcloud logging read 'resource.labels.container_name="<app-name>" AND resource.labels.pod_name="<pod-name>"'
```

Pipe the results to your output buffer of choice. 


You can also access logs through the gcloud console 
(Kubernetes -> Services & Ingress -> <app-name> -> <pod-name> -> "container logs")

### Ask for help!

If all else fails ask for help! 

The #uwit-cloudready channel is a great place to ask questions; folks are usually 
willing to help take a look at your configuration, and work with the results you've 
gotten so far.

If you suspect that the problem might be out of your control, reach out to help@uw.edu 
and describe the issue; you can note that the issue should be redirected to the 
Unix Engineering team.

[helm chart]: https://github.com/uw-iti-app-platform/helm-charts/tree/main/basic-web-service
[gcp-k8]: https://github.com/uwit-iam/gcp-k8
[ask for help!]: #ask-for-help
