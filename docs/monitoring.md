# Applicatoin Monitoring

Application monitoring in Kubernetes involves multiple components for a holistic picture.  The smallest unit is a an individual instance of a pod and it's own stdout.

However, pods come and go, so do the nodes that they run on and the cluster itself may need troubleshooting.

This readme is meant to evolve overtime as a gateway to resources, tools, tricks for monitoring applications.

## Prerequisite

1. Be a member of the `u_mciman_tenants_iam_conviewers` UW Group which should have Stackdriver viewer permissions in our GCP projects.

## Basics

- Browse to the GCP/GKE Console and click into the workload in question.

- You should now see links in the middle of the page for "Container Logs" as well as "Audit Logs".  You will also see CPU/Memory and Disk graphs.

- These links take you to GCP's Stackdriver where you can dive deper, create workspaces and alerts.

## Stackdriver

Stackdriver can be used in the GCP Console or command line.  Extensive [documentation and examples](https://cloud.google.com/monitoring/docs/) are available.

## kubectl

See the [logs](get-logs.md) readme