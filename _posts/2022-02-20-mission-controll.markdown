---
layout: post
title:  "Vault side car Pod Security Standards"
date:   2023-06-01 21:03:36 +0530
categories: Kubernetes Cloud Vault
---


Understanding Pod Security Standards in Kubernetes



Introduction:


Kubernetes has revolutionized the way we deploy and manage containerized applications at scale. As Kubernetes adoption continues to grow, ensuring the security of the underlying infrastructure becomes paramount. One critical aspect of securing Kubernetes deployments is implementing robust Pod Security Standards. In this blog, we will explore what Pod Security Standards are, why they are crucial, and how you can enforce them to bolster the security of your Kubernetes clusters.


What are Pod Security Standards?

Pod Security Standards, introduced in Kubernetes version 1.14, are a set of best practices and policies designed to enhance the security posture of pods, which are the basic units of deployment in Kubernetes. These standards enable cluster administrators to define and enforce security policies that pods must adhere to before they can run on the cluster.

The Pod Security Standards define three different policies to broadly cover the security spectrum. These policies are cumulative and range from highly-permissive to highly-restrictive. This guide outlines the requirements of each policy.

Profile	Description

 - Privileged:
Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.

 - Baseline:
Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.

 - Restricted:
Heavily restricted policy, following current Pod hardening best practices.



Why are Pod Security Standards Important?


Implementing Pod Security Standards is vital for several reasons:

1. Isolation: Pods running on a Kubernetes cluster should be isolated from each other to prevent unauthorized access and potential compromise. Pod Security Standards help enforce strong isolation by setting restrictions on network access, volume mounts, and other resources.

2. Attack Surface Reduction: By adhering to Pod Security Standards, you can reduce the attack surface exposed by pods. It helps mitigate the risk of vulnerabilities in containers or applications being exploited to gain unauthorized access to the host or other pods.

3. Compliance: Many industries and regulatory frameworks require strong security measures to protect sensitive data. Implementing Pod Security Standards can help you meet compliance requirements, ensuring the protection of customer data and maintaining trust.

4. Defense in Depth: A multi-layered security approach is crucial for any system. Pod Security Standards provide an additional layer of defense, complementing other security measures you may have implemented, such as network policies, RBAC, and container security scanning.


Enforcing Pod Security Standards:


To enforce Pod Security Standards in Kubernetes, you can leverage a built-in feature. 

Pod Security admission (PSA) is enabled by default in v1.23 and later, as it graduated to beta. Pod Security Admission is an admission controller that applies Pod Security Standards when pods are created. In this example, we will enforce the `restricted` Pod Security Standard, one namespace at a time.

Keep in mind you can also apply Pod Security Standards to multiple namespaces at once at the cluster level. 



Enable Pod Security Standards checking at the namespace namespace level

```
kubectl label --overwrite ns example \
   pod-security.kubernetes.io/warn=restricted \
   pod-security.kubernetes.io/warn-version=latest
```

deployment.yaml

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


Seccomp stands for secure computing mode and has been a feature of the Linux kernel since version 2.6.12. It can be used to sandbox the privileges of a process, restricting the calls it is able to make from userspace into the kernel. Kubernetes lets you automatically apply seccomp profiles loaded onto a node to your Pods and containers.

Allowed Values

 - RuntimeDefault
 - Localhost

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

Now that we have satisfied all the pod security standards for our orgchart app we can move on to our Vault side car.




[annotations](https://developer.hashicorp.com/vault/docs/platform/k8s/injector/annotations)

 -  vault.hashicorp.com/agent-json-patch - change the injected agent sidecar container using a JSON patch before it is created. This can be used to add, remove, or modify any attribute of the container. For example, setting this to [{"op": "replace", "path": "/name", "value": "different-name"}] will update the agent container's name to be different-name instead of the default vault-agent.

-  vault.hashicorp.com/agent-init-json-patch - same as vault.hashicorp.com/agent-json-patch, except that the JSON patch will be applied to the injected init container instead.

Conclusion:


Securing your Kubernetes deployments goes beyond configuring authentication and network policies. Implementing Pod Security Standards is a crucial step towards safeguarding your cluster against potential attacks and unauthorized access. By adhering to these standards, you can enhance the isolation and reduce the attack surface of pods, meet compliance requirements, and strengthen the overall security posture of your Kubernetes infrastructure. Embrace Pod Security Standards and make security an integral part of your Kubernetes deployment strategy.





