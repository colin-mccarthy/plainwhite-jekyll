---
layout: post
title:  "Mission Control as an abstration layer for Kubernetes"
date:   2020-02-24 21:03:36 +0530
categories: Kubernetes Cloud Mission-Control
---
Many Engineers I talk to are facing a challenge, they have been asked to move their apps and infrastructure to a particular cloud provider. They are often asking me about going fully GCP native or fully Azure native. A lot of the time they may have already gone down this path before with another cloud provider and are being asked to move now. They may have gone all in with AWS a year or two ago and now they need to move away from Amazon. This is a common trend I'm hearing all the time.
My question to them is this, if you asked again a year from now how quickly can you untangle yourself from that cloud provider and move your apps and clusters?

This is why I want to bring up the idea of an abstration layer for Kubernetes. Your developers should not need to care about where the clusters live as long as they have a kubernetes dial tone. Tanzu Mission Control gives you an interface to launch and manage many Kubernetes clusters in any or all of the major cloud providers and also cloud based or on prem Vsphere environments. You can create and delete clusters all from a single pane of glass using a SaaS based tool all from your web browser.

![Clusters]({{ site.url }}/plainwhite-jekyll/pics/test.png)




As you can see, here, I have a couple of different examples deployed, I have an AKs cluster, I have a vSphere cluster, I have, an Eks, cluster GCP clusters, I can manage all these clusters from one single pane of glass.

So if you want to have an autometed solution for deploying cluster this is that solution. It's automated and works on any cloud.

There's a concept of cluster groups within TMC. So cluster groups allow me to basically bucket these clusters into different groups. So for example, I can have a nonprofit group, I have a production cluster group, and then I can basically apply policies down to those different groups. So as you can see, I have a couple of groups here.

```
Group Pic
```

I can apply policies such as network policy, security policies, and Quota Management. If I didn't want my developers deploying applications from an unknown Container Registry outside of our network, I can prevent them from doing so. I can do a lot of things such as, image registry policies, I can block the latest tag, which is a best practice. So you can really control the behavior and enforce good practices within your environment.

This is pretty extensive, you also get a dashboard view of what's actually going on in this environment. As you can see, I have a couple of different policies that are being violated, especially my security policies here. So I can quickly glance at any bad actors within this environment, to see what actually is going on there.

```
policy pic
```



And you can set these as audit rules as well. So when you're first rollout TMC or you don't want to be too restrictive to your developers, you can set an audit setting that only alerts you when they're violating these policies. And that way, you can, have a nice conversation with the developer and say, this is how you should be doing things.


There are several different inspection tools that you can use. I can do scans such as a performance scan, the CIS benchmark, CIS benchmark is from the Center for Information Security. This is pretty much like all your best practices of how you should be configuring your Kubernetes clusters. You can basically run these scans periodically on your clusters to make sure they're aligning to best practices and principles.

```
scan pic
```



```
Cluster API
```

Tanzu Mission control uses an opesource project named Cluster API to handle the bootstrapping of Kubernetes clusters. Cluster API is part of an effort driven by a group known as SIG Cluster Lifecycle, SIG in this context stands for special interest group. And this is the group that is focused on streamlining and improving the Cluster Lifecycle Management for Kubernetes clusters, In an Open Source fashion. Cluster API provides Kubernetes style API's and patterns. These are declarative API's that allow you as a cluster operator, or cluster admin to declaratively define what a cluster should look like, and then have cluster API. Reconcile that or realize that declarative definition.

One of the goals of Cluster API, is to provide lifecycle management for Kubernetes clusters and work both on premises and in the public cloud, which means that users can experience a similar user experience, whether running clusters on premises environment like Vsphere or a public cloud environment.

I'd like to provide some definitions for frequently used terms. The first is a management cluster, a management cluster is a Kubernetes cluster and into this Kubernetes cluster have been installed the cluster API components. This enables this management cluster to manage the lifecycle of other Kubernetes clusters. These other Kubernetes clusters are properly known in cluster API, as workload clusters. 

To accomplish Lifecycle Management cluster API also uses the concept of a provider. Providers have names like cluster API provider for AWS, or cluster API provider for vSphere. These providers, provide the support for an integration with a particular infrastructure platform.

As a cluster admin you should be able to, create Kubernetes clusters by filling out a few fileds in Mission Contrlo and then have The Cluster API Management cluster in this case, actually go out and create the cluster and bootstrap it and get it running to your definition.

Tanzu Mission Control is a SaaS offering that gives you the ability to manage any Kubernetes cluster, regardless if it's in Vsphere, or not.  Mission Control is using Cluster API behind the scenes.




