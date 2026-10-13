# Kubernetes and Kubernetes Architecture — Complete Study Notes

## 1. What Is Kubernetes?

**Kubernetes (K8s)** is an open-source container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

Kubernetes was originally developed at Google and is now maintained by the Cloud Native Computing Foundation (CNCF).

### Why Do We Need Kubernetes?

Docker helps us build and run containers. However, managing hundreds or thousands of containers across multiple servers becomes difficult when done manually.

Kubernetes automates many of these operational tasks.

**Problems Kubernetes Solves:**

- Container deployment and management.
- Automatic scaling of applications.
- Self-healing when containers fail.
- Load balancing traffic across application instances.
- Service discovery between applications.
- Rolling updates and rollbacks.
- Resource management across servers.
- Configuration and secret management.
- Running applications across multiple worker nodes.

### Real-World Example

Imagine an e-commerce application with the following components:

- Frontend: React application.
- Backend: Java Spring Boot application.
- Database: PostgreSQL.
- Cache: Redis.

During normal traffic, you might run three backend replicas. During a festival sale, traffic may increase significantly, requiring more backend replicas.

Kubernetes can help scale the backend, distribute traffic, restart failed containers, and manage application deployment.

**Important:** Kubernetes does not automatically solve every availability or scaling problem. Proper configuration, resource limits, health checks, and monitoring are still necessary.

---

## 2. What Is Container Orchestration?

Container orchestration means automating the deployment, scheduling, networking, scaling, and management of containers.

### Without Kubernetes

An engineer may need to:

1. Log in to individual servers.
2. Start containers manually.
3. Check whether containers are running.
4. Restart failed containers.
5. Distribute traffic manually.
6. Add more containers during high traffic.
7. Update applications on multiple servers.
8. Roll back failed deployments manually.

### With Kubernetes

Kubernetes helps automate these tasks through declarative configuration and continuous reconciliation.

For example, you declare that an application should have three replicas. Kubernetes continuously works toward maintaining that desired state.

If one replica becomes unavailable, Kubernetes attempts to restore the desired number of replicas.

---

## 3. Kubernetes Architecture Overview

Kubernetes architecture consists of two major parts:

1. **Control Plane** — manages and coordinates the cluster.
2. **Worker Nodes** — run application workloads.

A Kubernetes cluster contains the control plane and one or more worker nodes. In managed Kubernetes services, the cloud provider manages some or all control-plane infrastructure.

### Architecture Diagram

```text
                     Developer / DevOps Engineer
                                |
                                v
                     kubectl / Kubernetes API
                                |
                                v
                 +-----------------------------+
                 |        CONTROL PLANE        |
                 |                             |
                 |  kube-apiserver             |
                 |  etcd                       |
                 |  kube-scheduler             |
                 |  kube-controller-manager    |
                 |  cloud-controller-manager   |
                 |  (when applicable)          |
                 +--------------+--------------+
                                |
                     Desired State / Scheduling
                                |
             +------------------+------------------+
             |                                     |
             v                                     v
    +--------------------+                +--------------------+
    |    WORKER NODE 1   |                |    WORKER NODE 2   |
    |                    |                |                    |
    | kubelet            |                | kubelet            |
    | kube-proxy*        |                | kube-proxy*        |
    | Container Runtime  |                | Container Runtime  |
    |                    |                |                    |
    | +----------------+ |                | +----------------+ |
    | | Pod: Frontend  | |                | | Pod: Backend   | |
    | +----------------+ |                | +----------------+ |
    | +----------------+ |                | +----------------+ |
    | | Pod: Backend   | |                | | Pod: Backend   | |
    | +----------------+ |                | +----------------+ |
    +--------------------+                +--------------------+

    * kube-proxy may be replaced by another networking
      implementation that provides Service networking.
```

### Main Idea

- The control plane decides and coordinates what should happen.
- Worker nodes run the application workloads.
- The API server is the main communication entry point.
- The scheduler selects suitable nodes for unscheduled Pods.
- Controllers continuously reconcile actual state with desired state.
- kubelet works with the container runtime to run the assigned Pods.

---

## 4. Control Plane Components

The control plane manages the overall Kubernetes cluster.

### 4.1 kube-apiserver

The API server is the central entry point for Kubernetes API operations.

It receives requests from tools such as `kubectl`, validates requests, performs authentication and authorization checks, and coordinates API operations.

**Responsibilities:**

- Exposes the Kubernetes API.
- Authenticates requests.
- Authorizes access using mechanisms such as RBAC.
- Performs admission control.
- Validates and processes Kubernetes objects.
- Coordinates persistence of cluster state in etcd.

**Example:**

```bash
kubectl get pods
```

The request goes to the API server. The API server retrieves the relevant information and returns the result.

When you create a Deployment:

```bash
kubectl apply -f deployment.yaml
```

The request is sent to the API server, which processes the configuration and persists the desired state.

**Important:** The API server is the main communication hub, but it does not directly run application containers.

### 4.2 etcd

etcd is a distributed key-value store used by Kubernetes to persist cluster state and configuration.

**Responsibilities:**

- Stores Kubernetes API object data.
- Persists desired and observed cluster state represented by API objects.
- Supports reliable state management through consensus.
- Provides the persistence layer used by the API server.

**Example:**

When you create a Deployment with three replicas, Kubernetes persists the Deployment specification and related state through the API server.

Controllers then work to make the actual cluster match the desired configuration.

**Important points:**

- etcd is critical to the control plane.
- Protect etcd with access controls, encryption where appropriate, and reliable backups.
- Losing etcd data without a usable backup can make cluster recovery difficult.
- Applications should generally not access etcd directly.

### 4.3 kube-scheduler

The scheduler selects an appropriate worker node for each newly created Pod that has not yet been assigned to a node.

**Responsibilities:**

- Evaluates available nodes.
- Considers CPU and memory requests.
- Applies node selectors and node affinity.
- Considers taints and tolerations.
- Respects topology and scheduling constraints.
- Selects a suitable node for a Pod.

**Example:**

Suppose a Pod requests:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
```

The scheduler looks for a node that can satisfy the Pod's scheduling requirements.

It does not simply select the node with the lowest CPU utilization. It considers resource requests and other scheduling constraints.

**Important:** The scheduler assigns the Pod to a node. The kubelet on that node then handles the Pod's execution.

### 4.4 kube-controller-manager

The controller manager runs controllers that continuously compare the desired state with the actual state.

**Examples of controllers:**

- Deployment-related controllers.
- ReplicaSet controller.
- Node controller.
- Job controller.
- EndpointSlice controller.

**Example:**

You configure a Deployment with three replicas.

```yaml
spec:
  replicas: 3
```

If only two replicas are available, the relevant controllers work to restore the desired number.

The controller manager does not directly start containers. It manages reconciliation through Kubernetes API objects.

**Key concept: Reconciliation loop**

```text
Read Desired State
        |
        v
Observe Actual State
        |
        v
Compare Both States
        |
        v
Take Corrective Action
        |
        v
Repeat Continuously
```

This reconciliation model is fundamental to Kubernetes.

### 4.5 cloud-controller-manager

The cloud controller manager integrates Kubernetes with supported cloud-provider functionality.

It may manage operations related to:

- Cloud load balancers.
- Cloud-backed node information.
- Cloud routes.
- Integration with cloud infrastructure.

Its exact responsibilities depend on the cloud provider and cluster configuration.

**Example in AWS:**

An AWS-integrated Kubernetes environment can provision an AWS load balancer for a suitable Kubernetes Service or Ingress configuration through the relevant controller integration.

**Important:** Not every Kubernetes cluster uses a cloud-controller-manager. On-premises and local clusters may use different infrastructure integrations.

---

## 5. Worker Node Components

Worker nodes provide the compute resources on which Pods run.

### 5.1 kubelet

kubelet is the node-level agent responsible for ensuring that the Pods assigned to its node are running as specified.

**Responsibilities:**

- Watches for Pods assigned to its node through the Kubernetes API.
- Works with the container runtime to create and manage containers.
- Reports node and Pod status.
- Runs configured container health probes.
- Coordinates volume mounting and other node-level Pod requirements.

**Example:**

The scheduler assigns a backend Pod to Worker Node 1.

The kubelet on Worker Node 1 works with the container runtime to start the Pod's containers.

If a container exits and its Pod restart policy requires a restart, kubelet works to restart it.

**Important:** kubelet is not the cluster scheduler and does not independently decide where a Pod should be placed.

### 5.2 Container Runtime

The container runtime is responsible for running containers.

Examples include:

- containerd
- CRI-O

The runtime communicates with kubelet through the Container Runtime Interface (CRI).

Docker Engine itself is not the standard runtime directly used by modern Kubernetes through CRI. Docker-built images can still be used in Kubernetes when they are compatible with the supported image format.

**Example flow:**

```text
kubelet
   |
   v
Container Runtime Interface (CRI)
   |
   v
containerd / CRI-O
   |
   v
Container Processes
```

**Important:** Docker is commonly used to build and test container images. Kubernetes can run those images using a compatible container runtime without requiring Docker Engine on every worker node.

### 5.3 kube-proxy

kube-proxy implements Kubernetes Service networking rules on nodes in many cluster configurations.

It commonly uses mechanisms such as iptables or IPVS, depending on the implementation and configuration.

**Responsibilities:**

- Helps implement virtual IP behavior for Services.
- Directs Service traffic toward eligible backend endpoints.
- Updates networking rules as Service endpoints change.

**Important:** kube-proxy is not mandatory in every cluster. Some networking implementations provide Service forwarding without kube-proxy.

### 5.4 Container Network Interface (CNI)

CNI is a specification and ecosystem for configuring container networking.

A Kubernetes cluster needs a networking solution that enables Pods to communicate according to the cluster's network model.

Examples of networking solutions include:

- Calico
- Cilium
- Flannel

A CNI plugin can provide Pod connectivity, and some implementations also provide network policy enforcement or additional networking features.

**Important:** CNI is not itself one specific networking product. Calico, Cilium, and Flannel are examples of implementations.

---

## 6. Important Kubernetes Objects

Kubernetes uses API objects to describe the desired state of workloads and infrastructure.

### 6.1 Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers that share certain resources, such as network identity and configured volumes.

**Key points:**

- A Pod receives an IP address in the cluster's Pod network.
- Containers within the same Pod can communicate through `localhost`.
- Containers in the same Pod can share mounted volumes.
- Pods are generally replaceable and should not be treated as permanent identities.
- Most application Pods are managed through higher-level controllers.

**Example:**

A backend application might run inside a Pod containing one application container.

### 6.2 ReplicaSet

A ReplicaSet maintains a specified number of matching Pod replicas.

For example:

```yaml
spec:
  replicas: 3
```

The ReplicaSet controller works to maintain three matching Pods.

In most application deployments, you create a Deployment rather than managing a ReplicaSet directly.

### 6.3 Deployment

A Deployment manages stateless application rollouts through ReplicaSets.

**Responsibilities:**

- Maintains the desired number of replicas through its managed resources.
- Supports rolling updates.
- Supports rollback to previous Deployment revisions when available.
- Manages changes to the application Pod template.

**Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: nginx:1.28
          ports:
            - containerPort: 80
```

This is a simple demonstration of Deployment structure. For a real backend, use the appropriate application image and configuration.

### 6.4 Service

A Service provides a stable network endpoint for accessing a group of eligible Pods.

Pod IP addresses may change when Pods are replaced. Clients should generally access the application through a Service rather than depending on individual Pod IPs.

**Service types:**

- **ClusterIP:** Exposes a Service internally within the cluster.
- **NodePort:** Exposes a Service through a port on eligible nodes.
- **LoadBalancer:** Requests an external load-balancing integration when supported.
- **ExternalName:** Maps a Service name to an external DNS name.

### 6.5 ConfigMap

A ConfigMap stores non-confidential configuration data.

Examples:

- Application environment settings.
- Feature flags.
- Non-secret configuration files.
- Service URLs.

Do not use a ConfigMap as a secure storage location for passwords or tokens.

### 6.6 Secret

A Secret stores sensitive configuration data, such as credentials and tokens.

**Security considerations:**

- Base64 encoding is not encryption.
- Configure encryption at rest where appropriate.
- Apply least-privilege RBAC permissions.
- Avoid exposing Secret values in logs or source repositories.
- Consider external secret-management systems for production.

### 6.7 Namespace

A Namespace logically separates groups of Kubernetes resources within a cluster.

Common examples:

- `development`
- `testing`
- `staging`
- `production`

Namespaces help with organization, access control, and resource quota management. They do not provide complete security isolation by themselves.

### 6.8 StatefulSet

A StatefulSet manages workloads that need stable Pod identities and ordered deployment or scaling behavior.

Typical examples include certain database and distributed-system deployments.

StatefulSets are often used with persistent storage, but they do not automatically make an application highly available or provide database replication.

### 6.9 DaemonSet

A DaemonSet ensures that a Pod runs on each eligible node, subject to its scheduling configuration.

Common use cases:

- Node-level monitoring agents.
- Log collection agents.
- Networking components.
- Security agents.

### 6.10 Job and CronJob

A Job runs a task to completion.

A CronJob creates Jobs according to a schedule.

Examples:

- Database migration.
- Data-processing task.
- Scheduled cleanup.
- Periodic report generation.

---

## 7. How Kubernetes Works: End-to-End Example

Suppose you want to deploy a web application with three replicas.

### Step 1: Create the Deployment Manifest

You write a YAML file that specifies the application image and desired replica count.

### Step 2: Apply the Configuration

```bash
kubectl apply -f deployment.yaml
```

kubectl sends the request to the Kubernetes API server.

### Step 3: Validate and Persist

The API server authenticates and authorizes the request, performs applicable validation and admission checks, and coordinates persistence.

### Step 4: Controllers Reconcile the State

The Deployment controller works to create or update a ReplicaSet. The ReplicaSet controller works to create the required Pods.

### Step 5: Scheduler Selects Nodes

The scheduler evaluates eligible nodes for each unscheduled Pod and assigns suitable nodes.

### Step 6: kubelet Starts the Workload

The kubelet on each selected node works with the container runtime to start the assigned Pod's containers.

### Step 7: Networking Is Configured

The cluster networking implementation provides Pod connectivity. Service networking makes the application accessible through a stable Service endpoint.

### Step 8: Application Becomes Available

Once the application is running and any required readiness checks succeed, the Pods can become eligible Service endpoints.

### Step 9: Kubernetes Continues Monitoring State

If a Pod becomes unavailable, controllers and node-level agents respond according to the relevant workload, health, and restart policies.

**Architecture flow:**

```text
deployment.yaml
      |
      v
kubectl apply
      |
      v
kube-apiserver
      |
      v
Persist Desired State in etcd
      |
      v
Deployment Controller
      |
      v
ReplicaSet Controller
      |
      v
Pod Objects Created
      |
      v
kube-scheduler
      |
      v
Worker Node Selected
      |
      v
kubelet + Container Runtime
      |
      v
Application Container Running
      |
      v
Service + Pod Networking
      |
      v
Application Traffic
```

The actual control-plane operations are asynchronous and may overlap. This diagram represents the conceptual flow rather than a strict sequence of synchronous calls.

---

## 8. Kubernetes Networking Architecture

Networking allows Pods, Services, and external clients to communicate.

### Pod Networking

Pods normally receive addresses from the cluster's Pod network. The networking implementation provides connectivity between Pods according to the cluster's configuration.

### Service Networking

A Service gives clients a stable endpoint that targets eligible Pods.

Example:

```text
Frontend Pod
     |
     v
backend-service
     |
     +----------+
     |          |
     v          v
Backend Pod 1  Backend Pod 2
```

The Service can distribute connections across eligible endpoints, subject to the networking implementation and traffic policies.

### DNS in Kubernetes

Kubernetes commonly provides cluster DNS through CoreDNS.

Applications can use Service DNS names instead of hardcoding Pod IP addresses.

For example:

```text
backend-service
```

From another Namespace, a fully qualified Service name may look like:

```text
backend-service.production.svc.cluster.local
```

The exact DNS search paths and cluster domain depend on cluster configuration.

### Ingress and Gateway

Ingress provides HTTP and HTTPS routing through an Ingress controller.

The Gateway API provides a more expressive set of resources for configuring traffic routing when supported by the chosen implementation.

Creating an Ingress object alone does not guarantee that traffic will be routed. An appropriate controller must be installed and configured.

---

## 9. Kubernetes Scaling and Self-Healing

### Horizontal Pod Autoscaler (HPA)

HPA adjusts the number of workload replicas based on configured metrics and policies.

For example, the number of backend replicas may increase when CPU utilization rises.

Metrics infrastructure and appropriate resource requests are important for CPU-based scaling.

### Vertical Pod Autoscaler (VPA)

VPA can recommend or adjust CPU and memory requests based on observed workload behavior, depending on its configuration and supported mode.

### Cluster Autoscaler

Cluster Autoscaler can add or remove nodes in supported environments when its scaling conditions are met.

**Remember the difference:**

- HPA: scales the number of workload replicas.
- VPA: adjusts or recommends resource requests for Pods.
- Cluster Autoscaler: scales cluster nodes.

These systems address different capacity problems and must be configured to work together appropriately.

### Self-Healing

Kubernetes supports self-healing through several mechanisms:

- Restarting containers according to restart policy.
- Recreating missing replicas through controllers.
- Replacing Pods managed by suitable controllers.
- Removing unready Pods from normal Service traffic.
- Detecting and responding to unhealthy nodes.

Self-healing does not guarantee that an application will recover from every failure. Application bugs, broken dependencies, insufficient capacity, and persistent-storage failures may require additional intervention.

---

## 10. Kubernetes Deployment Strategies

### Rolling Update

Gradually replaces old application Pods with new ones.

Benefits:

- Can reduce downtime.
- Allows controlled application updates.
- Supports rollback when a previous revision is available.

Important settings include:

- `maxSurge`
- `maxUnavailable`

### Recreate

Terminates existing Pods before creating replacement Pods. This can cause downtime and may be suitable when versions cannot safely coexist.

### Blue-Green Deployment

Maintains two application environments, such as blue and green, and switches traffic between them.

This approach typically requires additional capacity and an explicit traffic-switching mechanism.

### Canary Deployment

Gradually exposes a new application version to a subset of traffic or users.

Canary delivery generally requires traffic management, metrics, and an appropriate rollout mechanism; a standard Deployment alone does not provide sophisticated percentage-based traffic splitting.

---

## 11. Kubernetes Storage Architecture

Containers are replaceable, so data that must survive container or Pod replacement needs appropriate storage.

### PersistentVolume (PV)

Represents storage made available to a Kubernetes cluster.

### PersistentVolumeClaim (PVC)

A request for storage by a workload.

### StorageClass

Defines a class of storage and can enable dynamic provisioning through a supported storage provisioner.

### Basic Storage Flow

```text
Application Pod
      |
      v
PersistentVolumeClaim (PVC)
      |
      v
StorageClass / Provisioner
      |
      v
PersistentVolume (PV)
      |
      v
Underlying Storage
```

In dynamic provisioning, a PVC can trigger the creation of a PV through the configured provisioner.

Storage behavior depends on the storage driver, access modes, reclaim policy, and the application's own data-management design.

**Important:** Kubernetes persistent storage does not replace database backups, replication, or disaster-recovery planning.

---

## 12. Kubernetes Security Architecture

Security should be designed across the control plane, workloads, networking, and storage.

### Important Controls

- **Authentication:** Establishes who or what is making a request.
- **Authorization:** Determines which actions the requester is allowed to perform.
- **RBAC:** Defines role-based permissions.
- **ServiceAccount:** Provides an identity for workloads and automation.
- **NetworkPolicy:** Controls permitted Pod traffic when supported and enforced by the networking implementation.
- **Pod security controls:** Restrict risky container and Pod configurations.
- **Secrets management:** Protects credentials and tokens.
- **Image security:** Uses trusted images and vulnerability scanning.
- **Least privilege:** Minimizes permissions and container privileges.

### Example RBAC Principle

A monitoring application that only needs to read Pod information should not automatically receive permission to delete Deployments or modify Secrets.

Grant only the permissions needed for the workload.

---

## 13. Kubernetes Observability

Production Kubernetes requires visibility into cluster health and application behavior.

### Metrics

Common tools:

- Prometheus
- Grafana
- Metrics Server

Metrics help answer questions about CPU, memory, request rates, latency, and errors.

### Logging

Centralized logging systems collect application and infrastructure logs.

Examples include:

- Fluent Bit
- Loki
- Elasticsearch-based logging systems

### Tracing

Distributed tracing helps identify latency and failures across multiple services.

Examples include:

- OpenTelemetry
- Jaeger
- Grafana Tempo

### Common Troubleshooting Commands

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get services -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl get events -A --sort-by=.metadata.creationTimestamp
kubectl top nodes
kubectl top pods -A
```

`kubectl top` requires a functioning metrics source, commonly Metrics Server. The `--previous` option helps inspect logs from a previous container instance when available.

---

## 14. Common Kubernetes Pod Failures

### CrashLoopBackOff

The container repeatedly exits and Kubernetes delays subsequent restart attempts.

Possible causes:

- Application crashes.
- Incorrect startup configuration.
- Missing environment variables.
- Failed dependency connections.
- Incorrect commands or arguments.

Useful commands:

```bash
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

### ImagePullBackOff

Kubernetes cannot pull the requested image.

Possible causes:

- Incorrect image name or tag.
- Registry authentication failure.
- Network connectivity problems.
- Missing image-pull credentials.

### Pending

A Pod has not yet been scheduled or has not completed the steps needed to become running.

Possible causes:

- Insufficient CPU or memory.
- Taints without matching tolerations.
- Scheduling constraints.
- Unavailable persistent storage.

Check:

```bash
kubectl describe pod <pod-name>
kubectl get events -A
```

### OOMKilled

A container was terminated after exceeding its memory allowance or due to memory pressure and system-level memory management.

Check resource requests and limits, application memory use, and node pressure.

### Node NotReady

A node is not reporting the healthy state Kubernetes expects.

Investigate:

- kubelet status.
- Node resource pressure.
- Network connectivity.
- Container runtime health.
- Node infrastructure and cloud events.

---

## 15. Kubernetes vs Docker

| Feature | Docker / Container Tools | Kubernetes |
|---|---|---|
| Primary purpose | Build and run containers | Orchestrate containerized workloads |
| Image building | Commonly supported | Not its primary responsibility |
| Container execution | Supported by Docker Engine | Uses a supported container runtime |
| Multi-node scheduling | Not the main focus of Docker Engine | Core capability |
| Desired-state reconciliation | Limited to individual tool capabilities | Core Kubernetes model |
| Replica management | Requires additional orchestration for broad cluster management | Deployment and ReplicaSet controllers |
| Service discovery | Available through supported Docker networking features | Kubernetes Services and DNS |
| Automated scaling | Depends on surrounding tools | Supports workload and node autoscaling integrations |
| Rolling application updates | Depends on deployment tooling | Supported by Deployments |
| Self-healing | Limited to individual runtime features | Multiple controllers and node agents cooperate |

**Interview point:** Docker and Kubernetes are not direct replacements for each other. Container tools build and run containers, while Kubernetes orchestrates workloads across a cluster.

---

## 16. Kubernetes vs Traditional Server Deployment

### Traditional Deployment

An application is installed directly on a server or virtual machine.

Scaling may require creating additional servers, configuring the application, and updating traffic routing.

### Containerized Deployment

The application is packaged into a container image, providing a consistent runtime artifact.

### Kubernetes Deployment

The application is declared using Kubernetes objects, and the cluster coordinates scheduling, replicas, networking, and updates.

```text
Traditional:
Application -> Operating System -> Server

Container:
Application -> Container Image -> Container Runtime

Kubernetes:
Application Image -> Pod -> Worker Node
                         |
                         v
                 Kubernetes Cluster
```

Containers still use the host operating-system kernel. They are not full virtual machines.

---

## 17. Essential Kubernetes Commands

### Cluster Information

```bash
kubectl cluster-info
kubectl get nodes
kubectl get namespaces
kubectl version
```

### Pods

```bash
kubectl get pods
kubectl get pods -A
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```

### Deployments

```bash
kubectl get deployments
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=3
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

### Services

```bash
kubectl get services
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

### Apply and Delete Resources

```bash
kubectl apply -f deployment.yaml
kubectl get -f deployment.yaml
kubectl delete -f deployment.yaml
```

### Context and Namespace

```bash
kubectl config get-contexts
kubectl config current-context
kubectl get pods -n production
kubectl config set-context --current --namespace=production
```

Always verify the current context and Namespace before performing changes in a production cluster.

---

## 18. Practical Kubernetes Lab

### Objective

Deploy an NGINX application, scale it to three replicas, expose it internally, and inspect its status.

### Step 1: Create a Deployment

```bash
kubectl create deployment nginx --image=nginx
```

### Step 2: Scale to Three Replicas

```bash
kubectl scale deployment nginx --replicas=3
```

### Step 3: Verify Pods

```bash
kubectl get pods -o wide
```

### Step 4: Create a ClusterIP Service

```bash
kubectl expose deployment nginx --port=80 --type=ClusterIP
```

### Step 5: Inspect the Service

```bash
kubectl get services
kubectl describe service nginx
```

### Step 6: Check Deployment Health

```bash
kubectl rollout status deployment/nginx
kubectl get deployment nginx
```

### Step 7: Clean Up

```bash
kubectl delete service nginx
kubectl delete deployment nginx
```

**Expected learning outcome:** Understand how Deployments create Pods, how replicas are maintained, and how Services provide stable access to application Pods.

You need a working Kubernetes cluster, such as a suitable local cluster or a managed cluster, to perform this lab.

---

## 19. Kubernetes Interview Questions

### Q1. What is Kubernetes?

Kubernetes is an open-source container orchestration platform used to deploy, scale, manage, and recover containerized workloads.

### Q2. What are the main Kubernetes architecture components?

The main control-plane components are kube-apiserver, etcd, kube-scheduler, and kube-controller-manager. A cloud-controller-manager is used when applicable.

Worker nodes commonly run kubelet and a container runtime. Service networking may be implemented by kube-proxy or another networking solution.

### Q3. What is the role of the API server?

It is the main Kubernetes API entry point and handles request processing, authentication, authorization, validation, and coordination with persistent cluster state.

### Q4. What is the difference between a Pod and a Deployment?

A Pod is the smallest deployable unit. A Deployment manages application rollouts and uses ReplicaSets to maintain the desired number of Pod replicas.

### Q5. What is the difference between a ReplicaSet and a Deployment?

A ReplicaSet maintains matching Pod replicas. A Deployment manages ReplicaSets and provides declarative rollout and rollback functionality for stateless workloads.

### Q6. What happens when a Pod crashes?

The outcome depends on the Pod's restart policy and its managing controller. A container may be restarted, or a controller may create a replacement Pod to restore the desired state.

### Q7. What happens when a worker node fails?

The control plane detects node health problems. Depending on the failure and configuration, Pods may become unavailable and controllers may create replacement Pods on eligible nodes. Rescheduling takes time and depends on available capacity, storage constraints, and workload configuration.

### Q8. What is etcd?

etcd is the distributed key-value store used to persist Kubernetes cluster state.

### Q9. What is the difference between ClusterIP and NodePort?

ClusterIP exposes a Service internally within the cluster. NodePort exposes it through a port on eligible nodes.

### Q10. What is the difference between HPA and Cluster Autoscaler?

HPA scales workload replicas. Cluster Autoscaler changes the number of cluster nodes in supported environments.

### Q11. What is a Namespace?

A Namespace groups Kubernetes resources logically and can be used with access controls and resource quotas.

### Q12. What is the difference between ConfigMap and Secret?

ConfigMap stores non-confidential configuration. Secret stores sensitive configuration, but secure handling still requires appropriate access controls and encryption.

### Q13. What is a readiness probe?

A readiness probe checks whether a container is ready to serve traffic. A failed readiness probe normally makes the Pod unready, preventing it from being treated as a ready Service endpoint.

### Q14. What is a liveness probe?

A liveness probe checks whether a container should be restarted because it is unhealthy according to the configured probe.

### Q15. What is the difference between liveness and readiness?

Liveness helps determine whether a container should be restarted. Readiness determines whether a container should receive normal Service traffic.

---

## 20. Kubernetes Learning Roadmap for DevOps Engineers

Study Kubernetes in this order:

1. Container fundamentals and Docker.
2. Kubernetes architecture and cluster components.
3. Pods, labels, selectors, and Namespaces.
4. ReplicaSets and Deployments.
5. Services and cluster networking.
6. ConfigMaps and Secrets.
7. Requests, limits, and health probes.
8. Volumes, PVs, PVCs, and StorageClasses.
9. Ingress, Gateway API, and DNS.
10. RBAC, ServiceAccounts, and NetworkPolicies.
11. Helm and package management.
12. Kubernetes troubleshooting.
13. Prometheus and Grafana monitoring.
14. HPA and cluster autoscaling.
15. StatefulSets, DaemonSets, Jobs, and CronJobs.
16. Kubernetes security and production best practices.
17. GitOps with Argo CD.
18. Managed Kubernetes with Amazon EKS.
19. CI/CD integration and production deployment projects.

### Recommended Hands-On Projects

- Deploy a frontend and backend using separate Deployments.
- Connect frontend and backend through Services.
- Configure resource requests, limits, and health probes.
- Add ConfigMaps and Secrets.
- Deploy an application using Helm.
- Configure Ingress and HTTPS.
- Install monitoring and centralized logging.
- Deploy a sample application to Amazon EKS.
- Build a CI/CD pipeline that deploys an application to Kubernetes.
- Implement GitOps using Argo CD.

---

## 21. Final Revision Summary

- **Kubernetes:** Container orchestration platform.
- **Cluster:** Control plane plus worker nodes.
- **kube-apiserver:** Main API entry point.
- **etcd:** Persistent cluster-state store.
- **kube-scheduler:** Selects nodes for unscheduled Pods.
- **kube-controller-manager:** Runs reconciliation controllers.
- **kubelet:** Ensures assigned Pods run on its node.
- **Container runtime:** Runs containers.
- **kube-proxy:** Implements Service networking in many configurations.
- **CNI:** Networking specification and plugin ecosystem.
- **Pod:** Smallest deployable unit.
- **ReplicaSet:** Maintains Pod replicas.
- **Deployment:** Manages ReplicaSets and application rollouts.
- **Service:** Stable endpoint for accessing eligible Pods.
- **ConfigMap:** Non-confidential configuration.
- **Secret:** Sensitive configuration requiring secure handling.
- **Namespace:** Logical grouping of resources.
- **PV / PVC:** Persistent storage resource and storage request.
- **HPA:** Scales workload replicas.
- **Cluster Autoscaler:** Adjusts cluster node capacity in supported environments.
- **Ingress:** HTTP/HTTPS routing through an appropriate controller.
- **Prometheus and Grafana:** Common metrics and visualization tools.

**Core principle to remember:**

Kubernetes is a declarative system. You specify the desired state, and Kubernetes continuously works to reconcile the actual cluster state with that desired state.
