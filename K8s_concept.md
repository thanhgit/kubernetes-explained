# Main concept in kubernetes
- Kubernetes provides basic mechanism for deployment, maintainance and scaling of containerized applications
- It uses building blocks
    - To maintain the state requested by the user
    - Implementing the transaction from the current observable state to the requested state

![](./media/kubernetes_architecture.png)
## High level architecture
### Data node (data plane)
- Node are bare-metal server, on-premises VMs, or VMs on the cloud provider
- Each node contains:
    - Container runtime such as docker
    - Kubelet: is responsible for managing container
    - Kube-proxy: is responsible for network and load balancing

### Master node (control plane)
- Kubernetes cluster also contains one or more master nodes that run the kubernetes control plane
- It contains:
    - API server: provides JSON over HTTP API
    - Scheduler: select nodes to run container
    - Controller manager: run controllers
    - Distributed storage: such as etcd (a global variable configuration store)

### Kube-proxy
- Kube-proxy is a network proxy that runs on each node in the cluster, it allows the `service` to map traffic from one port to another
- It configures the Netfilter rules on all of the nodes according to the Service's definition in the API server. From kubernetes 1.9 onward it uses the `netlink interface` to create IPVS rules. These rules direct traffic to the appropriate Pod
- Source NAT is used for pods to communicate to external world
- IPtables are used extensively for load balancing and NAT

### Network control policy
- It controls communication between `Pods` and `Services`
- Network plugins like Calico, WeaveNet support network policies
 
#### Example 1: reviews service allow from product service
![](media/example1_network_control_policy.png)
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reviews-allow-from-product
spec:
  policyTypes:
  - Ingress
  podSelector:
    matchLabels:
      app: reviews
  ingress:
  - from:
    - podSelector:
      matchLabels:
        app: productpage
```

#### Example 2: reviews service prohibit all traffic
![](media/example2_network_control_policy.png)
```yaml
apiVersion: network.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: reviews-prohibit-all-traffic
spec:
  policyTypes:
  - Ingress
  podSelector:
    matchLabels:
      app: reviews
  ingress: []
```
### DNS
- Service discovery achieved by SkyDNS
- SkyDNS runs as a pod in the cluster
![](media/sky_dns.png)

## Kubernetes building block
### POD
- Every pod has a unique IP
- Share same namespace and volume, lifecycle => like "logical host"
- Containers within pod communicate with each other using standard inter-process communication like SystemV semaphores or POSIX share memory
- Use cases for multi-containers pods-sidecars, proxies/adapters

#### Single pod networking
![](media/single_pod_network.png)

#### Kubernetes networking overview
![](media/kubernetes_network_overview.png)

- L2 approach
![](media/l2_kubernetes_network.png)

- L3 approach
![](media/l3_kubernetes_network.png)

- Overlay approach
![](media/overlay_kubernetes_network.png)
### LABEL
- It is a key/value pair that is attached to kubernetes resource

### SELECTOR
- It is used to organize kubernetes resources that have label
- An equality-based selector defines a condition for selecting resources that have an specific label value
- A set-based selector defines a condition for selecting resources that have a label value within the specifed set of values

### CONTROLLER
- It manages a set of pods and ensures that the cluster is in the specified state
- Pods are managed by Replication controller (RC) are automatically replaced if they fail, get deleted or terminated
- Multiple types of controller such as replication controller (RC), deployment controller (DC), Replica set (RS)
    - #### `Replication controller`
        - It is responsible for running the specified number of pod copies (replicas) across the cluster
        - It only supports equality-based selectors
    - #### `Deployment controller`
        - A deployment defines a desired state for logical group of pods and replicas sets
        - It can be updated, rolled out and rolled back
        - It can be rolled back to an earlier revision if the current deployment is not stable
    - #### `Replica set`
        - It is the next generation of replication controller, It supports set-based selectors

### Service
- A service uses a selector to define a logical group of pods and defines a policy to access 
- Services provide a stable VIP, it automatically routes to backend pods
- VIP to backend pod mapping is managed by kube-proxy, implemented using iptables
- There are several types of services such as ClusterIP, NodePort and LoadBalancer
    - ClusterIP exposes pods in cluster (default)
    - NodePort exposes pods to external traffic by forwarding traffic from a `static port` on each node to the container port. Using `<NodeIP>:<NodePort>` to access
    - LoadBalancer exposes pods to external traffic as NodePort, however it also provides a load balancer
    - ExternalName: maps the service to the contents of the `externalname` field (`api.sewardnguye.com`), by returning a `CNAME` record with its value. No proxing of any kind is set up(kube-dns >= 1.7, coreDNS >= 0.0.8)
    ![](media/overview_kubernetes_network.png)
- Template
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
spec:
  selector:
    app: my-svc
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```
- Note: A service can map any `incomming port` to a `target port`

- Delve into `NodePort`
![](media/nodeport_kubernetes_network.png)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  labels:
    app: my-svc
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 9080
  selector:
    app: my-svc
```

- Delve into `LoadBalancer`
![](media/loadbalancer_kubernetes_network.png)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-svc
  labels:
    app: my-svc
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 9080
  selector:
    app: my-svc
```

- Delve into `Ingress Controller`
![](media/ingress_kubernetes_network.png)
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: gateway
spec:
  backend:
    serviceName: productpage
    servicePort: 9080
  rules:
  - host: mydomain.com
    http:
      paths:
      - path: /productpage
        backend:
          serviceName: productpage
          servicePort: 9080
      - path: /login
        backend:
          serviceName: productpage
          servicePort: 9080
      - path: /*
        backend:
          serviceName: productpage
          servicePort: 9080
```

### Volume
- A volume is defined at the pod level and is used to preserve data across container crashes
- It is used to share data between containers in a pod
- The same lifecycle as pod

#### Persistent volume (PV)
- It represents a real networked storage unit in a cluster
- It has lifecycle independent of any individual pod

### Persistent volume claim
- It defines a specific amount of storage requested and specific access modes

### JOB
- It is used to create one or more pods and ensures that a specified number of them successfully terminate
- It tracks the succcessful completions, and when a specified number of successful completions is reached, the job ifself is complete
- There are several types of jobs:
    - Non-parallel jobs
    - Parallel jobs with a work queue

### DAEMON SET
- It ensures that all or some nodes run a copy of a pod
- Use case: log collection daemon or a monitoring daemon on each node

### NAMESPACE
- It provides a logical partition of the cluster's resources 
- Different namespaces can be assigned different quatas for resource limitions

### QUOTA
- A quata sets resource limitations such as CPU, memory, number of pods or services, for a given namespace 

## Tips
### Using `labels` and `selectors` for fine-grained control
- Equality-based selector
```text
release: stable
environment: dev
```
- Set-based selector
```text
environment in (dev, test)
environment notin (live)
release = stable, environment = dev
```

### Service discovery
- Kubernetes supports finding a service in 2 ways: through EV or using DNS

