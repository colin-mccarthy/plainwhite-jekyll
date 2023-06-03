---
layout: post
title:  "Vault Kubernetes Clients"
date:   2023-06-4 21:03:36 +0530
categories: Kubernetes Cloud Vault
---


Understanding Vault Clients in Kubernetes

**Introduction:**

The Kubernetes auth method can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

The default endpoint is auth/kubernetes/login. If this auth method was enabled at a different path, use that value instead of kubernetes. [Docs here](https://developer.hashicorp.com/vault/docs/auth/kubernetes)

```yaml
curl \
    --request POST \
    --data '{"jwt": "<your service account jwt>", "role": "demo"}' \
    http://127.0.0.1:8200/v1/auth/kubernetes/login
```

The response will contain a token at auth.client_token:

```json
{
  "auth": {
    "client_token": "38fe9691-e623-7238-f618-c94d4e7bc674",
    "accessor": "78e87a38-84ed-2692-538f-ca8b9f400ab3",
    "policies": ["default"],
    "metadata": {
      "role": "demo",
      "service_account_name": "myapp",
      "service_account_namespace": "default",
      "service_account_secret_name": "myapp-token-pd21c",
      "service_account_uid": "aa9aa8ff-98d0-11e7-9bb7-0800276d99bf"
    },
    "lease_duration": 2764800,
    "renewable": true
  }
}
```





In a Kubernetes cluster, a UID (User ID) is a unique identifier assigned to each entity, including service accounts. When a service account is created, Kubernetes automatically generates a UID for it. The UID serves as a globally unique identifier for the service account within the cluster.

The process of creating a UID for a service account involves several steps:

1. API Server Request: When a service account is created in Kubernetes, an API server request is made to the Kubernetes API server. The request contains the necessary information to create the service account, such as the account name, namespace, and other metadata.

2. Admission Controller: The API server forwards the service account creation request to the admission controller. Admission controllers are plugins that intercept and process API requests before they are persisted in the cluster. They enforce security policies and perform various validations.

3. Service Account Controller: The admission controller interacts with the Service Account Controller, which manages service accounts within the cluster. The Service Account Controller generates a UID for the service account during this process.

4. UID Generation: The UID generation process typically involves creating a unique identifier using various algorithms, such as hashing or combining multiple attributes of the service account like name, namespace, creation timestamp, or a random value. The exact method can vary depending on the Kubernetes implementation or configuration.

5. Persisting the UID: Once the UID is generated, it is associated with the service account and stored in the cluster's etcd database. The etcd database is a distributed key-value store that Kubernetes uses to store its configuration data.

From this point forward, the UID serves as a unique identifier for the service account within the cluster. It is used for various purposes, such as access control, authentication, and resource allocation. Other components and features in the cluster, such as RBAC (Role-Based Access Control) and pod scheduling, may utilize the UID to identify and grant permissions to the service account.

**Openshift:**

OpenShift uses the Kubernetes platform as its foundation, so the process of creating a UID for a service account in OpenShift is similar to how it is done in Kubernetes.

When you create a service account in OpenShift, the system generates a unique UID (User ID) for that service account. The UID is a numeric identifier that is used to distinguish the service account from other entities within the cluster. It helps to maintain security and manage access control for resources.

The UID generation process in OpenShift typically involves the following steps:

1. When you create a service account, OpenShift generates a random, unique identifier for the account.
2. The UID is assigned to the service account and stored in the system's metadata associated with that account.
3. The UID is used to reference the service account when determining access permissions for various resources in the cluster.
4. OpenShift ensures that the generated UID is unique across the cluster to avoid any conflicts with other service accounts or users.

The specific implementation details of UID generation in OpenShift may vary based on the version and configuration of your OpenShift installation. It's worth noting that OpenShift builds upon Kubernetes, so the underlying concepts and mechanisms are shared between the two platforms.

**Spanning Kubernetes Namespaces:**

A Kubernetes service account is scoped to a single namespace and cannot span multiple namespaces. A service account provides an identity for processes running in a Kubernetes cluster. It is used to authenticate and authorize access to various Kubernetes resources within a specific namespace.

By default, a service account is created within a specific namespace, and it can only be used to access resources within that namespace. Each namespace has its own set of service accounts, and they are isolated from each other.

If you need to grant access to resources in multiple namespaces to a service account, you would need to create separate service accounts in each namespace and configure the necessary access permissions accordingly. This ensures proper access control and follows the principle of least privilege.

***Repaving Kubernetes Clusters: Ensuring Smooth Sailing for Your Container Orchestration***

As your Kubernetes clusters evolve and grow, it becomes crucial to maintain their health and performance. One essential practice in this regard is repaving, a process that involves rebuilding and rejuvenating your clusters. In this blog, we'll explore why repaving Kubernetes clusters is important, the benefits it offers, and the steps involved in carrying out this critical maintenance task.

Why Repave Kubernetes Clusters?

Over time, Kubernetes clusters can become cluttered with outdated components, configuration drift, security vulnerabilities, and accumulated technical debt. These issues can gradually degrade the stability, performance, and security of your cluster. Repaving addresses these challenges by resetting the cluster to a known, healthy state, ensuring that you start with a clean slate.

Improved Security and Compliance: Security vulnerabilities can emerge as software versions become outdated. By repaving Kubernetes clusters, you can update all components to their latest secure versions, reducing the risk of potential security breaches. Repaving also enables you to enforce compliance with security policies and best practices.

Automation and Infrastructure-as-Code: Leverage automation and Infrastructure-as-Code (IaC) tools like Terraform or Kubernetes manifests to define your cluster's desired state. Automating the provisioning and configuration process ensures consistency and repeatability.


**Vault Usage Metrics**

Recent enhancements

Starting in Vault 1.9, Vault changed the non-entity token computation logic to deduplicate non-entity tokens. For non-entity tokens (where there is no entity to which tokens map) Vault uses the contents of the token to generate a unique client identifier, based on the namespace ID and policies. The clientID will prevent the same token from being duplicated in the overall client count. Non-entity token tracking is done on access instead of creation. Since the change was made, Vault 1.10 (via the UI, API, documentation, etc.) refers to these non-entity tokens as non-entity clients, and unique entities as entity clients.


|Old term (v1.9 and earlier)| New term (v1.10 and later)|
| -----------               | -----------               |
| Total active clients      | Total clients             |
| Unique entities           | Entity clients            |
| Non-entity tokens         | Non-entity clients        |

Vault v1.10 enhanced the usage metrics dashboard so that you can select a billing period easier than before. In addition, you can view client count per auth mount. You can also view changes to clients month over month via the API.

Previously, if there is missing data within your billing period (e.g. data is available from January 2021 through October 2021, but April is missing), Vault returned metrics from May through October 2021, because that is the most recent contiguous set. As of Vault 1.11.0, Vault returns all available data within the specified billing period.

Vault 1.11 introduced Activity Export API allowing you to download the client count aggregated for the user-defined billing period. See the Export activity log using API section for more detail.

Also, refer to the Client Count [FAQ](https://developer.hashicorp.com/vault/docs/concepts/client-count/faq) which answers various client count questions.


**What is a client?**

Clients are unique applications, services, or users that authenticate to a HashiCorp Vault cluster. For billing and consumption, only unique and active clients during the billing period (monthly in the case of HCP and annual in the case of self-managed Vault) count towards totals. Each client is counted just once within a billing period, regardless of how many times it's been active. When a client authenticates to a cluster, those clients have unlimited access to that cluster for the remainder of the billing period. The client metric is a combination of active identity entities and active non-entity tokens. To learn more, refer to the documentation on What is a Client?.
