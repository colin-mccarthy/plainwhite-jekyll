---
layout: post
title:  "Mission Control as an abstration layer for Kubernetes"
date:   2020-02-24 21:03:36 +0530
categories: Kubernetes Cloud Mission-Control
---
Many Engineers I talk to are facing a challenge, they have been asked to move their apps and infrastructure to a particular cloud provider. They are often asking me about going fully GCP native or fully Azure native. A lot of the time they may have already gone down this path before with another cloud provider and are being asked to move now. They may have gone all in with AWS a year or two ago and now they need to move away from Amazon. This is a common trend I'm hearing all the time.
My question to them is this, if you asked again a year from now how quickly can you untangle yourself from that cloud provider and move your apps and clusters?

This is why I want to bring up the idea of an abstration layer for Kubernetes. Your developers should not need to care about where the clusters live as long as they have a kubernetes dial tone. Tanzu Mission Control gives you an interface to launch and manage Kubernetes cluster in all of the maajor cloud providers and also cloud based or on prem Vsphere environments. You can create and delete cluster all from a single pane of glass using a SaaS based tool all from you web browser.





![Clusters]({{ site.url }}/plainwhite-jekyll/pics/test.png)

![image](/plainwhite-jekyll/pics/test.png){: width="450" }

<img src="/plainwhite-jekyll/pics/test.png" alt="">

![image-title-here](/plainwhite-jekyll/pics/test.png){:class="img-responsive"}



