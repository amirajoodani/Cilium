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

# Step 1: Install the Cilium CLI Tool
Download and install the Cilium CLI to your local workstation.<br>

If installed correctly into your workstation’s executable path, you should be able to query the Cilium CLI tool for version information:<br>
```bash
cilium version
cilium-cli: v0.16.19 compiled with go1.23.1 on linux/amd64
cilium image (default): v1.16.2
cilium image (stable): v1.16.3
cilium image (running): unknown. Unable to obtain cilium version. Reason: release: not found 
```
We’ll install the default Cilium image into our prepared Kubernetes cluster. Note that the CLI tool also tells us we don’t yet have a Cilium installed in the cluster. We are going to fix that right now.<br>

# Step 2: Install Cilium
Now we can use the Cilium CLI tool to install Cilium:<br>
```bash
cilium install
🔮 Auto-detected Kubernetes kind: kind
ℹ️ Using Cilium version 1.16.2
🔮 Auto-detected cluster name: kind-kind
🔮 Auto-detected kube-proxy has been installed

It may take a couple of minutes for the installation process to complete. In another terminal, you can use the Cilium CLI tool to watch and wait for Cilium to be fully deployed and operational:

cilium status --wait

   /¯¯\
/¯¯\__/¯¯\    Cilium:          OK
\__/¯¯\__/    Operator:        OK
/¯¯\__/¯¯\    Envoy DaemonSet: OK
\__/¯¯\__/    Hubble Relay:    disabled
   \__/       ClusterMesh:     disabled 

DaemonSet           cilium           Desired: 3,    Ready: 3/3,   Available: 3/3
DaemonSet           cilium-envoy     Desired: 3,    Ready: 3/3,   Available: 3/3
Deployment          cilium-operator  Desired: 1,    Ready: 1/1,   Available: 1/1
Containers:         cilium           Running: 3
                    cilium-envoy     Running: 3
                    cilium-operator  Running: 1
Cluster Pods:       3/3 managed by Cilium
Helm chart version: 1.16.2
Image versions      cilium quay.io/cilium/cilium:v1.16.2@sha256:4386a8580d8d86934908eea022b0523f812e6a542f30a86a47edd8bed90d51ea: 3
                    cilium-envoy quay.io/cilium/cilium-envoy:v1.29.9-1726784081-a90146d13b4cd7d168d573396ccf2b3db5a3b047@sha256:9762041c3760de226a8b00cc12f27dacc28b7691ea926748f9b5c18862db503f: 3
                    cilium-operator quay.io/cilium/operator-generic:v1.16.2@sha256:cccfd3b886d52cb132c06acca8ca559f0fce91a6bd99016219b1a81fdbc4813a: 1
```
```bash
cilium hubble enable --ui
```
This command reconfigures and restarts the Cilium agents to ensure they have enabled the embedded Hubble services. The command will also install the cluster-wide Hubble components to enable cluster-wide network observability.<br>

You can verify Hubble components are ready using the cilium status command:<br>
```bash
cilium status

   /¯¯\
/¯¯\__/¯¯\     Cilium:          OK
\__/¯¯\__/     Operator:        OK
/¯¯\__/¯¯\     Envoy DaemonSet: OK
\__/¯¯\__/     Hubble Relay:    OK
   \__/        ClusterMesh:     disabled 

DaemonSet           cilium           Desired: 3,  Ready: 3/3,  Available: 3/3
DaemonSet           cilium-envoy     Desired: 3,  Ready: 3/3,  Available: 3/3
Deployment          cilium-operator  Desired: 1,  Ready: 1/1,  Available: 1/1
Deployment          hubble-relay     Desired: 1,  Ready: 1/1,  Available: 1/1
Deployment          hubble-ui        Desired: 1,  Ready: 1/1,  Available: 1/1
Containers:         cilium           Running: 3
                    cilium-envoy     Running: 3
                    cilium-operator  Running: 1
                    hubble-relay     Running: 1
                    hubble-ui        Running: 1
Cluster Pods:       5/5 managed by Cilium
Helm chart version: 1.16.2
Image versions      cilium quay.io/cilium/cilium:v1.16.2@sha256:4386a8580d8d86934908eea022b0523f812e6a542f30a86a47edd8bed90d51ea: 3
                    cilium-envoy quay.io/cilium/cilium-envoy:v1.29.9-1726784081-a90146d13b4cd7d168d573396ccf2b3db5a3b047@sha256:9762041c3760de226a8b00cc12f27dacc28b7691ea926748f9b5c18862db503f: 3
                    cilium-operator quay.io/cilium/operator-generic:v1.16.2@sha256:cccfd3b886d52cb132c06acca8ca559f0fce91a6bd99016219b1a81fdbc4813a: 1
                    hubble-relay quay.io/cilium/hubble-relay:v1.16.2@sha256:4b559907b378ac18af82541dafab430a857d94f1057f2598645624e6e7ea286c: 1
                    hubble-ui quay.io/cilium/hubble-ui-backend:v0.13.1@sha256:0e0eed917653441fded4e7cdb096b7be6a3bddded5a2dd10812a27b1fc6ed95b: 1
                    hubble-ui quay.io/cilium/hubble-ui:v0.13.1@sha25:e2e9313eb7caf64b0061d9da0efbdad59c6c461f6ca1752768942bfeda0796c6: 1
```

Step 3: Validate Cilium Operation
The Cilium CLI tool also provides a command to install a set of connectivity tests in a dedicated Kubernetes namespace. We can run these tests to validate that the Cilium install is fully operational:<br>
```bash
cilium connectivity test --request-timeout 30s --connect-timeout 10s
ℹ️ Monitor aggregation detected, will skip some flow validation steps
✨ [kind-kind] Creating namespace cilium-test for connectivity check...
✨ [kind-kind] Deploying echo-same-node service...
✨ [kind-kind] Deploying DNS test server configmap...
✨ [kind-kind] Deploying same-node deployment...
✨ [kind-kind] Deploying client deployment...
✨ [kind-kind] Deploying client2 deployment...
✨ [kind-kind] Deploying echo-other-node service...
✨ [kind-kind] Deploying other-node deployment...
⌛ [kind-kind] Waiting for deployments [client client2 echo-same-node] to become ready…
⌛ [kind-kind] Waiting for deployments [echo-other-node] to become ready...
...
✅ All 32 tests (239 actions) successful, 0 tests skipped, 1 scenarios skipped.
```
There are dozens of tests in the connectivity test suite, ensuring network and policy enforcement aspects work as expected. If you install a newer version of Cilium, the number of default connectivity tests might differ from what is depicted here. The connectivity tests may take additional time to complete the first time they are run, as they need to download container images required for the test deployments. The tests should take less time on subsequent runs as they use the locally cached images for the test deployments. You should expect the tests to take at least 10 minutes, allowing for extra time for the test environment container images to be downloaded the first time.<br>
<b>Note</b>:<br>
The connectivity tests require at least two worker nodes to deploy successfully in a cluster. The connectivity test pods will not be scheduled on nodes operating in the control-plane role. If you did not provision your cluster with two worker nodes, the connectivity test command may stall while waiting for the test environment deployments to complete.<br>

# Step 4: Examine Cluster with kubectl
With Cilium installed, we can use kubectl to confirm that the nodes are now ready and the required Cilium operational components are present in the cluster:<br>
```bash
kubectl get nodes
NAME                STATUS   ROLES          AGE  VERSION
kind-control-plane  Ready    control-plane  42m  v1.31.0
kind-worker         Ready    <none>         42m  v1.31.0
kind-worker2        Ready    <none>         42m  v1.31.0

kubectl get daemonsets --all-namespaces
NAMESPACE    NAME                   DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR                  AGE
cilium-test  host-netns             2       2       2     2          2                                        28m
cilium-test  host-netns-non-cilium  0       0       0     0          0         cilium.io/no-schedule=true     28m
kube-system  cilium                 3       3       3     3          3         kubernetes.io/os=linux         33m
kube-system  kube-proxy             3       3       3     3          3         kubernetes.io/os=linux         43m

kubectl get deployments --all-namespaces
NAMESPACE           NAME                    READY UP-TO-DATE AVAILABLE AGE
cilium-test         client                  1/1   1          1         29m
cilium-test         client2                 1/1   1          1         29m
cilium-test         echo-external-node      0/1   1          0         29m
cilium-test         echo-other-node         1/1   1          1         29m
cilium-test         echo-same-node          1/1   1          1         29m
kube-system         cilium-operator         1/1   1          1         33m
kube-system         coredns                 2/2   2.         2         44m
kube-system         hubble-relay            1/1   1          1         30m
kube-system         hubble-ui               1/1   1          1         30m
local-path-storage  local-path-provisioner  1/1   1          1         44m
```
You should find the cilium daemonset is running on all 3 nodes in the cluster, and the cilium-operator deployment is running on a single node.<br>
Congratulations! You have Cilium installed, providing connectivity in your Kubernetes cluster. Let’s take a moment and review exactly what you’ve just installed.<br>


# Operational Overview and Components
When you install Cilium, there are several operational components (some optional) that are installed. We’ll review the purpose of each of those in this section.<br>
<img width="728" height="660" alt="cilium-arch" src="https://github.com/user-attachments/assets/9c730352-f66c-48f3-bee5-9e35c90055ef" /> <br>

# Cilium Components
<b>Cilium Operator</b>:
The Cilium operator is responsible for managing duties in the cluster which should logically be handled once for the entire cluster, rather than once for each node in the cluster. The Cilium operator is not in the critical path for any forwarding or network policy decision. A cluster will generally continue to function if the operator is temporarily unavailable.<br>

<b>Cilium Agent</b>
The Cilium agent runs as a daemonset so that there is a Cilium agent pod running on every node in your Kubernetes cluster. The agent does the bulk of the work associated with Cilium:<br>

- Interacts with Kubernetes API server to synchronize cluster state.
- Interacts with the Linux kernel - loading eBPF programs and updating eBPF maps.
- Interacts with the Cilium CNI plugin executable, via a filesystem socket, to get notified of newly scheduled workloads.
- Creates on-demand DNS and Envoy proxies as needed based on requested network policy.
- Creates Hubble gRPC services when Hubble is enabled.<br>

<b>Cilium Client</b>
Each pod in the Cilium agent daemonset comes with a Cilium client executable that can be used to inspect the state of Cilium agent and eBPF maps resources installed on that node. The client communicates with the Cilium agent’s REST API from inside the daemonset pod.<br>
Note: This is not the same as the Cilium CLI tool executable that you installed on your workstation. The Cilium client executable is included in each Cilium agent pod, and can be used as a diagnostic tool to help troubleshoot Cilium agent operation if needed. You’ll seldom interact with the Cilium client as part of normal operation, but we’ll use it in some of the labs to help us see into the internals of the Cilium network state as we work with some of the Cilium capabilities.<br>

<b>Cilium CNI Plugin</b>
The Cilium agent daemonset also installs the Cilium CNI plugin executable into the Kubernetes host filesystem and reconfigures the node’s CNI to make use of the plugin. The CNI plugin executable is separate from the Cilium agent, and is installed as part of the agent daemonset initialization. When required, the Cilium CNI plugin will communicate with the running Cilium agent using a host filesystem socket.<br>

<b>Hubble Server</b>
The Hubble server runs on each node and retrieves the eBPF-based visibility from Cilium. It is embedded into the Cilium agent to achieve high performance and low overhead. It offers a gRPC service to retrieve flows and Prometheus metrics.<br>

<b>Hubble Relay</b>
When Hubble is enabled as part of a Cilium-managed cluster, the Cilium agents running on each node are restarted to enable the Hubble gRPC service to provide node-local observability. For cluster-wide observability, a Hubble Relay deployment is added to the cluster along with two additional services; the Hubble Observer service and the Hubble Peer service.The Hubble Relay deployment provides cluster-wide observability by acting as an intermediary between the cluster-wide Hubble Observer service and the Hubble gRPC services that each Cilium agent provides. The Hubble Peer service makes it possible for Hubble Relay to detect when new Hubble-enabled Cilium agents become active in the cluster. As a user, you will typically be interacting with the Hubble Observer service, using either the Hubble CLI tool or the Hubble UI, to gain insights into the network flows across your cluster that Hubble provides. <br>

<b>Hubble CLI & GUI</b>
The Hubble CLI (hubble) is a command line tool able to connect to either the gRPC API of hubble-relay or the local server to retrieve flow events.The graphical user interface (hubble-ui) utilizes relay-based visibility to provide a graphical service dependency and connectivity map.<br>

<b>Cluster Mesh API Server</b>
The Cluster Mesh API server is an optional deployment that is only installed if you enable the Cilium Cluster Mesh feature. Cilium Cluster Mesh allows Kubernetes services to be shared amongst multiple clusters.Cilium Cluster Mesh deploys an etcd key-value store in each cluster, to hold information about Cilium identities. It also exposes a proxy service for each of these etcd stores. Cilium agents running in any member of the same Cluster Mesh can use this service to read information about Cilium identity state globally across the mesh. This makes it possible to create and access global services that span the Cluster Mesh. Once the Cilium Cluster Mesh API service is available, Cilium agents running in any Kubernetes cluster that is a member of the Cluster Mesh are then able to securely read from each cluster’s etcd proxy thus gaining knowledge of Cilium identity state globally across the mesh. This makes it possible to create global services that span the cluster mesh. <br>

# Cilium Endpoints
Cilium makes application containers available on the network by assigning them IP addresses. All application containers which share a common IP address are grouped together in what Cilium refers to as an Endpoint. Endpoints are an internal representation that Cilium uses to efficiently manage container connectivity and will create Endpoints as needed for all containers it manages. And as it turns out, Kubernetes pods map directly to Cilium Endpoints, as Kubernetes pods are defined as a group of containers operating in a common set of Linux kernel namespaces and sharing an IP address. In a Cilium managed cluster, Cilium will create an Endpoint for each Kubernetes pod running in the cluster.<br>

# Cilium Identity
A key concept that makes Cilium work as efficiently as it does is Cilium’s notion of Identity. All Cilium Endpoints are assigned a label-based identity.<br>
<img width="1078" height="597" alt="cilium identity" src="https://github.com/user-attachments/assets/7401d330-5d6e-4c43-8475-e9fed38733e4" /><br>
A Cilium Identity is determined by Labels and is unique cluster-wide. An endpoint is assigned the identity which matches the endpoint’s Security Relevant Labels, i.e., all endpoints which share the same set of Security Relevant Labels will share the same identity. The unique numeric identifier associated with each identity is then used by eBPF programs in very fast lookup in the network datapath, and underpins how Hubble is able to provide Kubernetes-aware network observability.

As network packets enter or leave a node, Cilium’s eBPF programs map the source and destination IP address to the appropriate numeric identity identifier and then decide which datapath actions should be taken based on policy configuration referencing those numeric identifiers. Each Cilium agent is responsible for updating the identity-relevant eBPF maps with numeric identifiers relevant to endpoints running locally on the node by watching for updates to relevant Kubernetes resources.<br>


