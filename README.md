# What is Cilium ?
Cilium is an open source, cloud native solution for providing, securing, and observing network connectivity between workloads. In this course, we'll focus on learning how to use Cilium in Kubernetes environments where the aforementioned workloads are Kubernetes pods. However, it's important to point out that the benefits of Cilium aren't limited to Kubernetes environments. <br>
In a Kubernetes environment, Cilium acts as a networking plugin that provides connectivity between pods. It provides security by enforcing network policies and through transparent encryption, and the Hubble component of Cilium provides deep visibility into network traffic flows.<br>
Thanks to eBPF, Cilium’s networking, security, and observability logic can be programmed directly into the kernel, making Cilium and Hubble’s capabilities entirely transparent to application workloads. These will be containerized workloads in a Kubernetes cluster, though Cilium can also connect traditional workloads such as virtual machines and standard Linux processes.<br>

# Solving the Challenges of Container Networking at Scale
In the highly dynamic and complex world of microservices, thinking about networking primarily in terms of IP addresses and ports can lead to frustration. Using traditional networking tool implementations can be highly inefficient, providing only coarse-grained visibility and filtering and limiting your ability to troubleshoot and secure your container networks. From inception, Cilium was designed for large-scale, highly dynamic containerized environments. It natively understands container and Kubernetes identities and parses API protocols like HTTP, gRPC, and Kafka, providing visibility and security that is simpler and more powerful than a traditional firewall.<br>

# Built on Top of eBPF
Using eBPF as an underlying engine, Cilium creates a networking stack optimized for running microservices on platforms like Kubernetes. eBPF enables Cilium’s powerful security visibility and control logic to be dynamically inserted within the Linux kernel. eBPF makes the Linux kernel programmable so that applications like Cilium can hook into Linux kernel subsystems, bringing user space application context to kernel operations.Because eBPF runs inside the Linux kernel, Cilium security policies can be applied and updated without changing the application code or container configuration. eBPF programs hook into the Linux network datapath and can be used to drop packets based on network policy rules as the packet enters the network socket.<br>
<img width="1083" height="598" alt="ebpf" src="https://github.com/user-attachments/assets/9c9e046e-0709-46d7-a194-1b68edb9a07e" /> <br>
eBPF enables visibility into and control over systems and applications at a level of granularity and efficiency that was not possible before. It does so in a completely transparent way, without requiring the application to change. Cilium harnesses the power of eBPF by layering an efficient identity concept, bringing Kubernetes contextual information, like metadata labels, to eBPF-powered networking logic.<br>

# Networking
Cilium provides network connectivity, allowing communication between pods and other components (within or outside a Kubernetes cluster). It implements a simple flat Layer 3 network that can span multiple clusters and connect all application containers.By default, Cilium supports an overlay networking model, where a virtual network spans all hosts. Traffic in an overlay network is encapsulated for transport between different hosts. This mode is chosen as the default because it requires minimal infrastructure and integration and only IP connectivity between hosts.Cilium also offers the option of a native routing networking model, using the regular routing table on each host to route traffic to pod (or external) IP addresses. This mode is for advanced users and requires awareness of the underlying networking infrastructure. It works well with native IPv6 networks, cloud network routers, or pre-existing routing daemons.<br>

# Identity-aware Network Policy Enforcement
Network policies define which workloads are permitted to communicate with one another, securing your deployment by preventing unexpected traffic. Cilium can enforce both native Kubernetes NetworkPolicies, and an enhanced CiliumNetworkPolicy resource type.Traditional firewalls secure workloads by filtering on IP addresses and destination ports. In a Kubernetes environment, this requires manipulating the firewalls (or iptables rules) on all node hosts whenever a pod is started anywhere in the cluster to rebuild firewall rules corresponding to desired network policy enforcement. This doesn’t scale well.To avoid this situation, Cilium assigns an identity to groups of application containers based on relevant metadata, such as Kubernetes labels. The identity is then associated with all network packets emitted by the application containers, allowing eBPF programs to efficiently validate the identity at the receiving node – without using any Linux firewall rules. For example, when a deployment is scaled up, and a new pod is created somewhere in the cluster, the new pod shares the same identity as the existing pods. The eBPF program rules corresponding to network policy enforcement do not have to be updated again, as they already know about the pod’s identity! <br>
While traditional firewalls operate at Layers 3 and 4, Cilium can also secure modern Layer 7 application protocols such as REST/HTTP, gRPC, and Kafka (in addition to enforcing at Layers 3 and 4). It provides the ability to enforce network policy corresponding to application protocol request conditions such as: <br>

Allow all HTTP requests with method GET and path /public/.*. Deny all other requests.
Require the HTTP header X-Token: [0-9]+ to be present in all REST calls.

# Transparent Encryption
In-flight data encryption between services is now a requirement in many regulation frameworks such as PCI or HIPAA. Cilium supports simple-to-configure transparent encryption, using IPSec or WireGuard, that when enabled, secures traffic between nodes without requiring reconfiguring any workload .<br>


# Multi-cluster Networking
Cilium’s Cluster Mesh capabilities make it easy for workloads to communicate with services hosted in different Kubernetes clusters. You can make services highly available by running them in clusters in different regions, using Cilium Cluster Mesh to connect them . <br>

# Load Balancing
Cilium implements distributed load balancing for traffic between application containers and external services. As you’ll see later in this course, Cilium can fully replace components such as kube-proxy and be used as a standalone load balancer. Load balancing is implemented in eBPF using efficient hash tables, allowing for almost unlimited scale. <br>

# Enhanced Network Observability
While we’ve learned to love tools like tcpdump and ping, which will always find a special place in our hearts, they are just not up to the task of troubleshooting networking issues in dynamic Kubernetes cluster environments. Cilium strives to provide observability tooling that lets you quickly identify and fix cluster networking problems.Towards that end, Cilium includes a dedicated network observability component called Hubble. Hubble makes use of Cilium’s identity concept to make it easy to filter traffic in an actionable way and provides: <br>
Visibility into network traffic at Layer 3/4 (IP address and port) and Layer 7 (API Protocol).<br>
Event monitoring with metadata: When a packet is dropped, the tool reports not only the source and destination IP but also the full label information of both the sender and receiver, among other information.<br>
Configurable Prometheus metrics exports.<br>
A graphical UI to visualize the network traffic flowing through your clusters.<br>
<img width="1043" height="531" alt="hubble" src="https://github.com/user-attachments/assets/4faf7e01-f141-44c2-a106-acee43cf8b3c" /> <br>

# Prometheus Metrics
Cilium and Hubble export metrics about network performance and latency via Prometheus so that you can integrate Cilium metrics into your existing dashboards.<br>
<img width="1085" height="531" alt="cilium-metrcis" src="https://github.com/user-attachments/assets/8d9d2ca2-f7bb-41d9-aa84-b6eb898337b1" /> <br>

# Service Mesh
You’ve already seen that Cilium supports load-balancing between services, application layer visibility, and a variety of security-related features, all of which are features of a Kubernetes service mesh. Cilium also supports both Kubernetes Ingress and Gateway API to provide a full suite of service mesh features without requiring the overhead of sidecar containers injected into every pod.<br>

# Installation Methods
Cilium supports two methods of installation:<br>
Cilium CLI tool <br>
The CLI tool makes it easy to get started with Cilium, especially when you’re first learning about it. It uses the Kubernetes API directly to examine the cluster corresponding to an existing kubectl context and choose appropriate installation options for the Kubernetes implementation detected. We’ll use the Cilium CLI install method for most of the labs in the course.<br>
Helm chart <br>
The Helm chart method is meant for advanced installation and production environments where you want granular control of your Cilium installation. You must manually select the best datapath and IPAM mode for your particular Kubernetes environment. You can learn more about the Helm chart installation method in the Cilium documentation resources. We’ll use the Helm chart install method in a later chapter when getting familiar with some advanced capabilities.<br>

# Step 0: Prepare Your Kubernetes Cluster
Note: <br>
Skip this step if you are planning on using a Kubernetes cluster other than a local kind cluster for this lab. This step is here as a convenience to ensure everyone following this lab has a cluster environment readily available to work with. Docker is a prerequisite for setting up the kind cluster. Install Docker and set up the kind cluster.<br>

Download and install Docker <br>

Download and install kind (version >= v0.7.0) <br>

Here is the YAML configuration file for a 3-node kind cluster with default CNI disabled. Save this locally to your workstation as kind-config.yaml with the contents:<br>
```bash
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
networking:
  disableDefaultCNI: true
```
Now create a new kind cluster using this configuration:<br>
```bash
kind create cluster --config=kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.31.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-kind"
```
You can now use your cluster with: <br>
```bash
kubectl cluster-info --context kind-kind 
```
Have a nice day! 👋 <br>
Kind will create the cluster and will configure an associated kubectl context. Confirm your new kind cluster is the default kubectl context:<br>

kubectl config current-context
kind-kind

Now you should be able to use kubectl and the Cilium CLI tool and interact with your newly minted kind cluster.<br>

Note: Because you have created the cluster without a default CNI, the Kubernetes nodes are in a NotReady state:<br>
```bash
kubectl get nodes
NAME                STATUS    ROLES          AGE    VERSION
kind-control-plane  NotReady  control-plane  8m30s   v1.31.0
kind-worker         NotReady  <none>         8m17s   v1.31.0
kind-worker2        NotReady  <none>         8m17s   v1.31.0
```





