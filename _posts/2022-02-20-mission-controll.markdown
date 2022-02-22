---
layout: post
title:  "Mission Control as an abstration layer for Kubernetes cluster managment"
date:   2022-02-22 21:03:36 +0530
categories: Kubernetes Cloud Mission-Control
---


Many Engineers I talk to nowadays are facing a challenge, they have been asked to move their companies apps and infrastructure to a particular cloud provider. They are often asking me about the downside to going fully GCP native or, fully Azure native etc.. A lot of the time they may have already gone down this path before with another cloud provider, and now are being asked to move yet again. For example, they may have gone all in with AWS a year or two ago and now need to move away from Amazon and go into Azure. My question to them is this, if you need to move again a year from now, how quickly can you untangle yourself from that cloud provider and move your apps and clusters? 

This is why I want to bring up the idea of an abstration layer for Kubernetes Cluster lifecyle managment. VMware's Tanzu Mission Control gives you an interface to launch and manage your Kubernetes clusters in any or all of the major cloud providers and also on prem Vsphere environments. You can create and delete clusters from a single pane of glass using a SaaS based tool and all from your web browser.

Tanzu Mission Control uses VMware's flavor of Kubernetes called Tanzu Kubernetres Grid or TKG for short. TKG gives you the consistent Kubernetes dial tone no matter what cloud you are deploying your clusters into. You dont need to build the compute instances beforehand, there are no scripts that need to be run or command line tools needed, everyting is bootstrapped for you. You simply log into your Mission Control instance and deploy your clusters from there.

![Clusters]({{ site.url }}/plainwhite-jekyll/pics/test.png)

As you can see I have a couple examples of TKG clusters deployed into different clouds, I have a TKG clusters in AWS, a TKG cluster in vSphere, and others in Google Cloud ans Azure. I can manage the lifecycle of all these clusters from one single pane of glass, using one single flavor of Kubernetes. Giving you a consistent Kubernetes dial tone no matter what cloud your operating in.

So in the event you need to move to a new cloud provider it's as easy as inputting your cloud provider account details and launching a new TKG cluster. Think of the freedom this provides you to quickly pivot to a new cloud or a hybrid cloud model. Let's also think about your leverage for future contract renewals if the cloud provider's prices go up or your discounted pricing runs out, the dreaded vendor lock in scenario.



```
Creation: 
```


Let's look at how Mission control is able to do all of this behind the scenes. Mission control uses an opesource project named Cluster API to handle the bootstrapping of Kubernetes clusters. Cluster API is part of an effort driven by a group known as SIG Cluster Lifecycle. Cluster API provides Kubernetes style declarative API's that allow you to define what a cluster should look like. This allows Tanzu Mission Control to manage the creation, managment and then some time later the eventual deletion of your clusters. To accomplish Lifecycle Management cluster API uses the concept of a provider. These providers, allow the support for an integration with a particular infrastructure platform. You can think of these much like how provides work in something like Terrafrom, or how modules work in Ansible. As a cluster admin you should be able to create Kubernetes clusters by filling out a few fields in Mission Control, and then have Mission Control actually go out and create the cluster and bootstrap it and get it running to your definition.


```
Management:
```

There's a concept of cluster groups within Mission Control. So cluster groups allow me to basically bucket these clusters into different groups. So for example, You can have a nonprofit group, I have a production cluster group, and then I can basically apply policies down to those different groups. You can apply policies such as network policy, security policies, and Quota Management. If you didn't want developers deploying applications from an unknown Container Registry outside of our network, You can prevent them from doing so. You can do a lot of things such as, image registry policies, You can block the latest tag, which is a best practice. So you can really control the behavior and enforce good practices within your environment.

This is pretty extensive, you also get a dashboard view of what's actually going on in this environment. As you can see, I have a couple of different policies that are being violated, especially my security policies here. So I can quickly glance at any bad actors within this environment, to see what actually is going on there.


And you can set these as audit rules as well. So when you're first rollout Mission Control or you don't want to be too restrictive to your developers, you can set an audit setting that only alerts you when they're violating these policies. And that way, you can, have a nice conversation with the developer and say, this is how you should be doing things.


There are several different inspection tools that you can use. You can do scans such as a performance scan, the CIS benchmark, CIS benchmark is from the Center for Information Security. This is pretty much like all your best practices of how you should be configuring your Kubernetes clusters. You can basically run these scans periodically on your clusters to make sure they're aligning to best practices and principles.




```
Eventual deletion:
```







I'd like to provide some definitions for frequently used terms. The first is a management cluster, a management cluster is a Kubernetes cluster and into this Kubernetes cluster have been installed the cluster API components. This enables this management cluster to manage the lifecycle of other Kubernetes clusters. These other Kubernetes clusters are properly known in cluster API, as workload clusters. 



```
Abstration layer:
```

Your developers don't care about where the clusters live as long as they have a Kubernetes dial tone to launch their apps on. You shouldn't need to care what cloud provider your using as long as you have a Cloud dial tone to launch your clusters on.

Cluster API helps provides lifecycle management for Kubernetes clusters and works both on premises and in the public cloud, which means that users can experience a similar user experience, whether running clusters on premises environment like Vsphere or in a public cloud environment.







