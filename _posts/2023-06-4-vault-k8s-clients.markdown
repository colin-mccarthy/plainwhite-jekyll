---
layout: post
title:  "Vault Kubernetes Clients"
date:   2023-06-4 21:03:36 +0530
categories: Kubernetes Cloud Vault
---


Understanding Vault Clients in Kubernetes

**Introduction:**

The kubernetes auth method can be used to authenticate with Vault using a Kubernetes Service Account Token. This method of authentication makes it easy to introduce a Vault token into a Kubernetes Pod.

The default endpoint is auth/kubernetes/login. If this auth method was enabled at a different path, use that value instead of kubernetes.

```yaml
curl \
    --request POST \
    --data '{"jwt": "<your service account jwt>", "role": "demo"}' \
    http://127.0.0.1:8200/v1/auth/kubernetes/login
```

The response will contain a token at auth.client_token:

```JSON
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
