---
layout: post
title: "Installing the Kubernetes dashboard in IBM Cloud private"
date: 2017-09-15
---

IBM Cloud private is a relatively new project from IBM which will
provide an awesome private cloud environment to run containers,
managed by Kubernetes. There's way more to come in this space
over the next few months so keep your eyes peeled for further
developments.

I've been playing around with the new 2.1beta image on my laptop
using Vagrant and I went to spin up a Kubernetes dashboard to check
on some metrics of my cluster. I found the following message when I
opened by browser

```
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "endpoints \"kubernetes-dashboard\" not found",
  "reason": "NotFound",
  "details": {
    "name": "kubernetes-dashboard",
    "kind": "endpoints"
  },
  "code": 404
}
```

It seems the `kubernetes-dashboard` is not running as standard in ICp
(IBM Cloud private, it wouldn't be a technology product without a handy
acronym would it!?).

My first thought was that this might be tricky to install, I'd have to
find the correct version of the dashboard for the version I was running etc.
but I found some basic instructions and thought I'd give them a shot.
It worked, first time!

So here's what you need to do

1. Configure `kubectl` to point to your kube cluster. In ICp you can click
on the avatar on the top right of the UI and copy the commands into a terminal
and run them. This sets the context of `kubectl` to point to the ICp cluster.
2. Create the deployment, service and service account and cluster role binding
for `kubernetes-dashboard` by running
`kubectl create -f create -f https://git.io/kube-dashboard`
3. Go and grab a cup of coffee while the image is downloaded and the container
is created. If your cluster has a decent network connection you might only
have to wait 30 seconds or so. You can check on it's progress by monitoring the
status of pods in the `kube-system` namespace by running
`kubectl get pods --namespace kube-system`
4. Back from your coffee and suitably refreshed, you're ready to open up the dashboard.
`kubectl proxy`
5. Open a browser to `http://127.0.0.1:8001/ui`

Enjoy!
