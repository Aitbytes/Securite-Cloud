---
title: Glossary
---
 Let's dive deeper into some of the concepts related to Kubernetes that appear in [[attack|our scenario]], ensuring you have a clear understanding of their roles and significance.

### Understanding Services and Ports
In Kubernetes, services are abstractions that define a logical set of pods and a policy by which to access them. Services are essential for ensuring that applications inside a Kubernetes cluster can communicate with one another, despite changes in pod IP addresses.

- **Nmap:** A network scanning tool used to discover hosts and services on a computer network. In our scenario, it helps identify which ports are open, indicating available services.
- **Ports:** These are communication endpoints. A specific port number is associated with each service, allowing traffic to be directed appropriately.

### The Kubernetes API
The Kubernetes API is the interface used by external tools to interact with the cluster. It's available at a particular endpoint, such as port 6443, and provides operations for managing and configuring Kubernetes resources.

- **Kubectl:** A command-line tool that interfaces with the Kubernetes API, allowing you to execute commands against Kubernetes clusters. Think of it as your primary interface with Kubernetes.

### Kubelet
The Kubelet is a key component in Kubernetes, running on each node in the cluster. It ensures that containers are running in pods as expected. The Kubelet listens for instructions from the Kubernetes control plane and manages pods and containers based on those instructions.

### Service Accounts and Tokens
Service accounts in Kubernetes allow processes, rather than human users, to authenticate against the API. They contain credentials used for authenticating with the Kubernetes API.

- **Tokens:** These are keys associated with service accounts, granting authenticated access to perform operations according to the permissions assigned to the service account.

### Privileged Escalation
Privilege escalation refers to exploiting a bug, design flaw, or configuration oversight in an operating system or software application to gain elevated access to resources that are normally protected. In Kubernetes, ensuring that only necessary permissions are granted is crucial for maintaining security.

### Understanding RoleBindings and ClusterRoleBindings
These are objects used to define which users or service accounts have access to what resources within a Kubernetes cluster:

- **RoleBindings:** Link a Role to a user or group within a specific namespace, determining what actions they're permitted to perform.
- **ClusterRoleBindings:** Similar to RoleBindings, but they apply across the entire cluster, providing broader access rights.

### Pods and Containers
Pods are the smallest deployable units in Kubernetes, representing a group of one or more containers that share storage and network resources. Containers are the standardized packages that encapsulate an application's code and its dependencies, allowing it to run reliably across different computing environments.

By understanding these concepts, you gain a clearer picture of how Kubernetes orchestrates application deployments and manages underlying resources efficiently and securely. 
