---
layout: post
title:  "Vault Agent Sidecar Pod Security Standards"
date:   2023-05-27 21:03:36 +0530
categories: Kubernetes Cloud Vault
---


Understanding Pod Security Standards in Kubernetes



**Introduction:**


Kubernetes has revolutionized the way we deploy and manage containerized applications at scale. As Kubernetes adoption continues to grow, ensuring the security of the underlying infrastructure becomes paramount. One critical aspect of securing Kubernetes deployments is implementing robust Pod Security Standards. In this blog, we will explore what Pod Security Standards are, why they are crucial, and how you can enforce them to bolster the security of your Kubernetes clusters.


**What are Pod Security Standards?**

Pod Security Standards, are a set of best practices and policies designed to enhance the security posture of pods, which are the basic units of deployment in Kubernetes. These standards enable cluster administrators to define and enforce security policies that pods must adhere to before they can run on the cluster.

The Pod Security Standards define three different policies to broadly cover the security spectrum. These policies are cumulative and range from highly-permissive to highly-restrictive. [This guide outlines the requirements of each policy](https://kubernetes.io/docs/concepts/security/pod-security-standards/).

Profile	Description

 - <span style="color:red">**Privileged**</span>:
Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.

 - <span style="color:red">**Baseline**</span>:
Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.

 - <span style="color:red">**Restricted**</span>:
Heavily restricted policy, following current Pod hardening best practices.





**Why are Pod Security Standards Important?**


Implementing Pod Security Standards is vital for several reasons:

1. Isolation: Pods running on a Kubernetes cluster should be isolated from each other to prevent unauthorized access and potential compromise. Pod Security Standards help enforce strong isolation by setting restrictions on network access, volume mounts, and other resources.

2. Attack Surface Reduction: By adhering to Pod Security Standards, you can reduce the attack surface exposed by pods. It helps mitigate the risk of vulnerabilities in containers or applications being exploited to gain unauthorized access to the host or other pods.

3. Compliance: Many industries and regulatory frameworks require strong security measures to protect sensitive data. Implementing Pod Security Standards can help you meet compliance requirements, ensuring the protection of customer data and maintaining trust.

4. Defense in Depth: A multi-layered security approach is crucial for any system. Pod Security Standards provide an additional layer of defense, complementing other security measures you may have implemented, such as network policies, RBAC, and container security scanning.


**Enforcing Pod Security Standards:**


To enforce Pod Security Standards in Kubernetes, you can leverage a built-in feature. 

[Pod Security admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) (PSA) is enabled by default in v1.23 and later, as it graduated to beta. Pod Security Admission is an admission controller that applies Pod Security Standards when pods are created. In this example, we will enforce the **restricted** Pod Security Standard on a single namespace (test-ns).

Keep in mind you can also apply Pod Security Standards to multiple namespaces at once at the cluster level. 



 [Enable Pod Security Standards](https://kubernetes.io/docs/tutorials/security/ns-level-pss/) checking at the namespace level

```
kubectl label --overwrite ns test-ns \
   pod-security.kubernetes.io/warn=restricted \
   pod-security.kubernetes.io/warn-version=latest
```

Here is a vanilla deployment of an orgchart app that has been configured to use the Vault agent sidecar. The tutorial for this deployment can be found [here](https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-sidecar)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orgchart
  labels:
    app: orgchart
spec:
  selector:
    matchLabels:
      app: orgchart
  replicas: 1
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "internal-app"
        vault.hashicorp.com/agent-inject-secret-database-config.txt: "internal/data/database/config"
        vault.hashicorp.com/agent-json-patch: '[{"op": "replace", "path": "/securityContext/seccompProfile", "value": {"type": "RuntimeDefault"}}]'
      labels:
        app: orgchart
    spec:
      serviceAccountName: internal-app
      containers:
        - name: orgchart
          image: jweissig/app:0.0.1
 ```
 
 This deployment would be blocked as it does not pass the requirements of the restricted pod security standards mentioned above.
 
 **Testing**

You can use a --dry-run command to test if any warnings are present on your namespace before applying the restricted policy.

```
kubectl label --dry-run=server --overwrite ns --all \
pod-security.kubernetes.io/enforce=restricted
```


when testing with my vanilla deployment I saw the following violations.


```
Warning: orgchart-78b559df9c-7zr2w: allowPrivilegeEscalation != false, 
unrestricted capabilities, runAsNonRoot != true, seccompProfile
namespace/test-ns labeled (server dry run)
```

I saw the warnings start to go away once I modified the manifest with all the security settings listed below. The final seccompProfile warning went away when I added the annotations. This --dry-run command is very helpful for seeing exactly what changes to the manifest will remove what warnings. Once you have no warnings you can feel comfortable in deploying the manifest on a cluster/namespace configured with a restricted policy.
 
 
 
 

Here is the new deployment manifest with the changes made to satisfy the [restricted pod security standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted):


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: orgchart
  labels:
    app: vault-agent-injector-demo
spec:
  selector:
    matchLabels:
      app: vault-agent-injector-demo
  replicas: 1
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "internal-app"
        vault.hashicorp.com/agent-inject-secret-database-config.txt: "internal/data/database/config"
        vault.hashicorp.com/agent-json-patch: '[{"op": "replace", "path": "/securityContext/seccompProfile", "value": {"type": "RuntimeDefault"}}]'
        vault.hashicorp.com/agent-init-json-patch: '[{"op": "replace", "path": "/securityContext/seccompProfile", "value": {"type": "RuntimeDefault"}}]'
      labels:
        app: vault-agent-injector-demo
    spec:
      serviceAccountName: internal-app
      containers:
        - name: orgchart
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
            runAsNonRoot: true
            runAsUser: 10000
            seccompProfile:
              type: RuntimeDefault
          image: jweissig/app:0.0.1
```

Let's cover what changed and why.

Privilege Escalation:
Privilege escalation (such as via set-user-ID or set-group-ID file mode) should not be allowed.

```yaml
          securityContext:
            allowPrivilegeEscalation: false
```

Capabilities (v1.22+):	
Containers must drop ALL capabilities, and are only permitted to add back the NET_BIND_SERVICE capability.

```yaml
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
```

Running as Non-root:	
Containers must be required to run as non-root users.

```yaml
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
            runAsNonRoot: true
```
 
Running as Non-root user (v1.23+):	
Containers must not set runAsUser to 0

```yaml
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
            runAsNonRoot: true
            runAsUser: 10000
```

Seccomp (v1.19+):
Seccomp profile must be explicitly set to one of the allowed values. Both the Unconfined profile and the absence of a profile are prohibited

```yaml
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
            runAsNonRoot: true
            runAsUser: 10000
            seccompProfile:
              type: RuntimeDefault
```

[Seccomp](https://kubernetes.io/docs/tutorials/security/seccomp/) stands for secure computing mode and has been a feature of the Linux kernel since version 2.6.12. It can be used to sandbox the privileges of a process, restricting the calls it is able to make from userspace into the kernel. Kubernetes lets you automatically apply seccomp profiles loaded onto a node to your Pods and containers.

Allowed Values

 - RuntimeDefault
 - Localhost

Now that we have satisfied all the pod security standards for our orgchart app we can move on to our Vault agent sidecar.


We will use Vault agent [annotations](https://developer.hashicorp.com/vault/docs/platform/k8s/injector/annotations) to change the seccompProfile to the allowed profile of RuntimeDefault.

 - <span style="color:red">**vault.hashicorp.com/agent-json-patch**</span> - change the injected agent sidecar container using a JSON patch before it is created. This can be used to add, remove, or modify any attribute of the container. For example, setting this to [{"op": "replace", "path": "/name", "value": "different-name"}] will update the agent container's name to be different-name instead of the default vault-agent.

- <span style="color:red">**vault.hashicorp.com/agent-init-json-patch**</span> - same as vault.hashicorp.com/agent-json-patch, except that the JSON patch will be applied to the injected init container instead.

```yaml
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "internal-app"
        vault.hashicorp.com/agent-inject-secret-database-config.txt: "internal/data/database/config"
        vault.hashicorp.com/agent-json-patch: '[{"op": "replace", "path": "/securityContext/seccompProfile", "value": {"type": "RuntimeDefault"}}]'
        vault.hashicorp.com/agent-init-json-patch: '[{"op": "replace", "path": "/securityContext/seccompProfile", "value": {"type": "RuntimeDefault"}}]'
```

You will notice we added both **agent-json-patch** and **agent-init-json-patch**. This is becasue when the deployment spins up it will use both of these containers not just the sidecar. There is also an init container that you must be aware of as seen below. Failure to use both would cause the deployment to be blocked by the Pod Security Admission controller.

```
colin@colin-PKPF4HHXVY work % k get events -n test-ns
LAST SEEN   TYPE     REASON              OBJECT                          MESSAGE
31s         Normal   Scheduled           pod/orgchart-8cf6b6574-7kbdn    Successfully assigned test-ns/orgchart-8cf6b6574-7kbdn to vault-worker
31s         Normal   Pulled              pod/orgchart-8cf6b6574-7kbdn    Container image "hashicorp/vault:1.13.1" already present on machine
31s         Normal   Created             pod/orgchart-8cf6b6574-7kbdn    Created container vault-agent-init
31s         Normal   Started             pod/orgchart-8cf6b6574-7kbdn    Started container vault-agent-init
30s         Normal   Pulled              pod/orgchart-8cf6b6574-7kbdn    Container image "jweissig/app:0.0.1" already present on machine
30s         Normal   Created             pod/orgchart-8cf6b6574-7kbdn    Created container orgchart
30s         Normal   Started             pod/orgchart-8cf6b6574-7kbdn    Started container orgchart
30s         Normal   Pulled              pod/orgchart-8cf6b6574-7kbdn    Container image "hashicorp/vault:1.13.1" already present on machine
30s         Normal   Created             pod/orgchart-8cf6b6574-7kbdn    Created container vault-agent
30s         Normal   Started             pod/orgchart-8cf6b6574-7kbdn    Started container vault-agent
31s         Normal   SuccessfulCreate    replicaset/orgchart-8cf6b6574   Created pod: orgchart-8cf6b6574-7kbdn
31s         Normal   ScalingReplicaSet   deployment/orgchart             Scaled up replica set orgchart-8cf6b6574 to 1
```




**Conclusion:**


Securing your Kubernetes deployments goes beyond managing secrets, configuring authentication, and network policies. Implementing Pod Security Standards is a crucial step towards safeguarding your cluster against potential attacks and unauthorized access. By adhering to these standards, you can enhance the isolation and reduce the attack surface of pods, meet compliance requirements, and strengthen the overall security posture of your Kubernetes infrastructure. Embrace Pod Security Standards and make security an integral part of your Kubernetes deployment strategy.





