# First contact with `kubectl`

- `kubectl` is (almost) the only tool we'll need to talk to Kubernetes

- It is a rich CLI tool around the Kubernetes API

  (Everything you can do with `kubectl`, you can do directly with the API)

- On our machines, there is a `~/.kube/config` file with:

  - the Kubernetes API address

  - the path to our TLS certificates used to authenticate

- You can also use the `--kubeconfig` flag to pass a config file

- Or directly `--server`, `--user`, etc.

---
## `kubectl get`

- Let's look at our `Node` resources with `kubectl get`!

.lab[

- Look at the composition of our cluster:
  ```bash
	pi@pi-red:~ $ kubectl get node
	NAME       STATUS   ROLES           AGE    VERSION
	pi-black   Ready    <none>          9m3s   v1.33.2
	pi-red     Ready    control-plane   10m    v1.33.2
  ```

- These commands are equivalent:
  ```bash
	pi@pi-red:~ $ kubectl get no
	NAME       STATUS   ROLES           AGE     VERSION
	pi-black   Ready    <none>          9m46s   v1.33.2
	pi-red     Ready    control-plane   10m     v1.33.2

	pi@pi-red:~ $ kubectl get nodes
	NAME       STATUS   ROLES           AGE   VERSION
	pi-black   Ready    <none>          10m   v1.33.2
	pi-red     Ready    control-plane   11m   v1.33.2
  ```

---

## Obtaining machine-readable output

- `kubectl get` can output JSON, YAML, or be directly formatted

- Give us more info about the nodes:
  ```bash
	pi@pi-red:~ $   kubectl get nodes -o wide
	NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION       CONTAINER-RUNTIME
	pi-black   Ready    <none>          10m   v1.33.2   192.168.0.101   <none>        Debian GNU/Linux 12 (bookworm)   6.12.25+rpt-rpi-v8   containerd://1.7.27
	pi-red     Ready    control-plane   11m   v1.33.2   192.168.0.152   <none>        Debian GNU/Linux 12 (bookworm)   6.6.51+rpt-rpi-v8    containerd://1.7.27
  ```

- Let's have some YAML:
  ```bash
  kubectl get no -o yaml
  too much output
  kind: List
  ```
  See that `kind: List` at the end? It's the type of our result!

---

## (Ab)using `kubectl` and `jq`

- It's super easy to build custom reports


- Show the capacity of all our nodes as a stream of JSON objects:
  ```bash
	pi@pi-red:~ $ kubectl get nodes -o json | jq ".items[] | {name:.metadata name} + .status.capacity"

	{
	"name": "pi-black",
	"cpu": "4",
	"ephemeral-storage": "14725268Ki",
	"memory": "3888568Ki",
	"pods": "110"
	}
	{
	"name": "pi-red",
	"cpu": "4",
	"ephemeral-storage": "14729364Ki",
	"memory": "3882992Ki",
	"pods": "110"
	}
  ```

]


## Exploring types and definitions

- We can list all available resource types by running `kubectl api-resources`

```bash
pi@pi-red:~ $ kubectl api-resources
NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
bindings                                         v1                                true         Binding
componentstatuses                   cs           v1                                false        ComponentStatus
configmaps                          cm           v1                                true         ConfigMap
endpoints                           ep           v1                                true         Endpoints
events                              ev           v1                                true         Event
limitranges                         limits       v1                                true         LimitRange
namespaces                          ns           v1                                false        Namespace
nodes                               no           v1                                false        Node
persistentvolumeclaims              pvc          v1                                true         PersistentVolumeClaim
persistentvolumes                   pv           v1                                false        PersistentVolume
pods                                po           v1                                true         Pod
podtemplates                                     v1                                true         PodTemplate
replicationcontrollers              rc           v1                                true         ReplicationController
resourcequotas                      quota        v1                                true         ResourceQuota
secrets                                          v1                                true         Secret
serviceaccounts                     sa           v1                                true         ServiceAccount
services                            svc          v1                                true         Service
mutatingwebhookconfigurations                    admissionregistration.k8s.io/v1   false        MutatingWebhookConfiguration
validatingadmissionpolicies                      admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                admissionregistration.k8s.io/v1   false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                  admissionregistration.k8s.io/v1   false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1           false        CustomResourceDefinition
apiservices                                      apiregistration.k8s.io/v1         false        APIService
controllerrevisions                              apps/v1                           true         ControllerRevision
daemonsets                          ds           apps/v1                           true         DaemonSet
deployments                         deploy       apps/v1                           true         Deployment
replicasets                         rs           apps/v1                           true         ReplicaSet
statefulsets                        sts          apps/v1                           true         StatefulSet
selfsubjectreviews                               authentication.k8s.io/v1          false        SelfSubjectReview
tokenreviews                                     authentication.k8s.io/v1          false        TokenReview
localsubjectaccessreviews                        authorization.k8s.io/v1           true         LocalSubjectAccessReview
selfsubjectaccessreviews                         authorization.k8s.io/v1           false        SelfSubjectAccessReview
selfsubjectrulesreviews                          authorization.k8s.io/v1           false        SelfSubjectRulesReview
subjectaccessreviews                             authorization.k8s.io/v1           false        SubjectAccessReview
horizontalpodautoscalers            hpa          autoscaling/v2                    true         HorizontalPodAutoscaler
cronjobs                            cj           batch/v1                          true         CronJob
jobs                                             batch/v1                          true         Job
certificatesigningrequests          csr          certificates.k8s.io/v1            false        CertificateSigningRequest
leases                                           coordination.k8s.io/v1            true         Lease
endpointslices                                   discovery.k8s.io/v1               true         EndpointSlice
events                              ev           events.k8s.io/v1                  true         Event
flowschemas                                      flowcontrol.apiserver.k8s.io/v1   false        FlowSchema
prioritylevelconfigurations                      flowcontrol.apiserver.k8s.io/v1   false        PriorityLevelConfiguration
ingressclasses                                   networking.k8s.io/v1              false        IngressClass
ingresses                           ing          networking.k8s.io/v1              true         Ingress
ipaddresses                         ip           networking.k8s.io/v1              false        IPAddress
networkpolicies                     netpol       networking.k8s.io/v1              true         NetworkPolicy
servicecidrs                                     networking.k8s.io/v1              false        ServiceCIDR
runtimeclasses                                   node.k8s.io/v1                    false        RuntimeClass
poddisruptionbudgets                pdb          policy/v1                         true         PodDisruptionBudget
clusterrolebindings                              rbac.authorization.k8s.io/v1      false        ClusterRoleBinding
clusterroles                                     rbac.authorization.k8s.io/v1      false        ClusterRole
rolebindings                                     rbac.authorization.k8s.io/v1      true         RoleBinding
roles                                            rbac.authorization.k8s.io/v1      true         Role
priorityclasses                     pc           scheduling.k8s.io/v1              false        PriorityClass
csidrivers                                       storage.k8s.io/v1                 false        CSIDriver
csinodes                                         storage.k8s.io/v1                 false        CSINode
csistoragecapacities                             storage.k8s.io/v1                 true         CSIStorageCapacity
storageclasses                      sc           storage.k8s.io/v1                 false        StorageClass
volumeattachments                                storage.k8s.io/v1                 false        VolumeAttachment
```

- We can view the definition for a resource type:
  ```bash
  	kubectl explain <type>

	pi@pi-red:~ $ kubectl explain pod
	KIND:       Pod
	VERSION:    v1

	DESCRIPTION:
		Pod is a collection of containers that can run on a host. This resource is
		created by clients and scheduled onto hosts.
		
	FIELDS:
	apiVersion    <string>
		APIVersion defines the versioned schema of this representation of an object.
		Servers should convert recognized schemas to the latest internal value, and
		may reject unrecognized values. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

	kind  <string>
		Kind is a string value representing the REST resource this object
		represents. Servers may infer this from the endpoint the client submits
		requests to. Cannot be updated. In CamelCase. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

	metadata      <ObjectMeta>
		Standard object's metadata. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

	spec  <PodSpec>
		Specification of the desired behavior of the pod. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

	status        <PodStatus>
		Most recently observed status of the pod. This data may not be up to date.
		Populated by the system. Read-only. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status
  ```

- We can view the definition of a field in a resource, for instance:
  ```bash
	pi@pi-red:~ $ kubectl explain pod.kind
	KIND:       Pod
	VERSION:    v1

	FIELD: kind <string>


	DESCRIPTION:
		Kind is a string value representing the REST resource this object
		represents. Servers may infer this from the endpoint the client submits
		requests to. Cannot be updated. In CamelCase. More info:
		https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
  ```

- Or get the full definition of all fields and sub-fields:
  ```bash
	pi@pi-red:~ $ kubectl explain node --recursive
	KIND:       Node
	VERSION:    v1

	DESCRIPTION:
		Node is a worker node in Kubernetes. Each node will have a unique identifier
		in the cache (i.e. in etcd).
		
	FIELDS:
	apiVersion    <string>
	kind  <string>
	metadata      <ObjectMeta>
		annotations <map[string]string>
		creationTimestamp   <string>
		deletionGracePeriodSeconds  <integer>
		deletionTimestamp   <string>
		finalizers  <[]string>
		generateName        <string>
		generation  <integer>
		labels      <map[string]string>
		managedFields       <[]ManagedFieldsEntry>
		apiVersion        <string>
		fieldsType        <string>
		fieldsV1  <FieldsV1>
		manager   <string>
		operation <string>
		subresource       <string>
		time      <string>
		name        <string>
		namespace   <string>
		ownerReferences     <[]OwnerReference>
		apiVersion        <string> -required-
		blockOwnerDeletion        <boolean>
		controller        <boolean>
		kind      <string> -required-
		name      <string> -required-
		uid       <string> -required-
		resourceVersion     <string>
		selfLink    <string>
		uid <string>
	spec  <NodeSpec>
		configSource        <NodeConfigSource>
		configMap <ConfigMapNodeConfigSource>
			kubeletConfigKey        <string> -required-
			name    <string> -required-
			namespace       <string> -required-
			resourceVersion <string>
			uid     <string>
		externalID  <string>
		podCIDR     <string>
		podCIDRs    <[]string>
		providerID  <string>
		taints      <[]Taint>
		effect    <string> -required-
		enum: NoExecute, NoSchedule, PreferNoSchedule
		key       <string> -required-
		timeAdded <string>
		value     <string>
		unschedulable       <boolean>
	status        <NodeStatus>
		addresses   <[]NodeAddress>
		address   <string> -required-
		type      <string> -required-
		allocatable <map[string]Quantity>
		capacity    <map[string]Quantity>
		conditions  <[]NodeCondition>
		lastHeartbeatTime <string>
		lastTransitionTime        <string>
		message   <string>
		reason    <string>
		status    <string> -required-
		type      <string> -required-
		config      <NodeConfigStatus>
		active    <NodeConfigSource>
			configMap       <ConfigMapNodeConfigSource>
			kubeletConfigKey      <string> -required-
			name  <string> -required-
			namespace     <string> -required-
			resourceVersion       <string>
			uid   <string>
		assigned  <NodeConfigSource>
			configMap       <ConfigMapNodeConfigSource>
			kubeletConfigKey      <string> -required-
			name  <string> -required-
			namespace     <string> -required-
			resourceVersion       <string>
			uid   <string>
		error     <string>
		lastKnownGood     <NodeConfigSource>
			configMap       <ConfigMapNodeConfigSource>
			kubeletConfigKey      <string> -required-
			name  <string> -required-
			namespace     <string> -required-
			resourceVersion       <string>
			uid   <string>
		daemonEndpoints     <NodeDaemonEndpoints>
		kubeletEndpoint   <DaemonEndpoint>
			Port    <integer> -required-
		features    <NodeFeatures>
		supplementalGroupsPolicy  <boolean>
		images      <[]ContainerImage>
		names     <[]string>
		sizeBytes <integer>
		nodeInfo    <NodeSystemInfo>
		architecture      <string> -required-
		bootID    <string> -required-
		containerRuntimeVersion   <string> -required-
		kernelVersion     <string> -required-
		kubeProxyVersion  <string> -required-
		kubeletVersion    <string> -required-
		machineID <string> -required-
		operatingSystem   <string> -required-
		osImage   <string> -required-
		swap      <NodeSwapStatus>
			capacity        <integer>
		systemUUID        <string> -required-
		phase       <string>
		enum: Pending, Running, Terminated
		runtimeHandlers     <[]NodeRuntimeHandler>
		features  <NodeRuntimeHandlerFeatures>
			recursiveReadOnlyMounts <boolean>
			userNamespaces  <boolean>
		name      <string>
		volumesAttached     <[]AttachedVolume>
		devicePath        <string> -required-
		name      <string> -required-
		volumesInUse        <[]string>

  ```

## Type names

- The most common resource names have three forms:

  - singular (e.g. `node`, `service`, `deployment`)

  - plural (e.g. `nodes`, `services`, `deployments`)

  - short (e.g. `no`, `svc`, `deploy`)

- Some resources do not have a short name

- `Endpoints` only have a plural form

  (because even a single `Endpoints` resource is actually a list of endpoints)

## Viewing details

- We can use `kubectl get -o yaml` to see all available details

- However, YAML output is often simultaneously too much and not enough

- For instance, `kubectl get node node1 -o yaml` is:

  - too much information (e.g.: list of images available on this node)

  - not enough information (e.g.: doesn't show pods running on this node)

  - difficult to read for a human operator

- For a comprehensive overview, we can use `kubectl describe` instead

---

## `kubectl describe`

- `kubectl describe` needs a resource type and (optionally) a resource name

- It is possible to provide a resource name *prefix*

  (all matching objects will be displayed)

- `kubectl describe` will retrieve some extra information about the resource

.lab[

- Look at the information available for `pod` with one of the following commands:
  ```bash
	pi@pi-red:~ $ kubectl describe pod pingpong
	Name:             pingpong
	Namespace:        default
	Priority:         0
	Service Account:  default
	Node:             pi-black/192.168.0.101
	Start Time:       Thu, 10 Jul 2025 22:00:14 +0530
	Labels:           run=pingpong
	Annotations:      <none>
	Status:           Running
	IP:               10.90.1.4
	IPs:
	IP:  10.90.1.4
	Containers:
	pingpong:
		Container ID:  containerd://a6f55b150fcb28bbbaac71a6d567d08309d6d5f92d557afbd975c420254a3329
		Image:         alpine
		Image ID:      docker.io/library/alpine@sha256:8a1f59ffb675680d47db6337b49d22281a139e9d709335b492be023728e11715
		Port:          <none>
		Host Port:     <none>
		Command:
		ping
		127.0.0.1
		State:          Running
		Started:      Thu, 10 Jul 2025 22:00:16 +0530
		Ready:          True
		Restart Count:  0
		Environment:    <none>
		Mounts:
		/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-h8bs6 (ro)
	Conditions:
	Type                        Status
	PodReadyToStartContainers   True 
	Initialized                 True 
	Ready                       True 
	ContainersReady             True 
	PodScheduled                True 
	Volumes:
	kube-api-access-h8bs6:
		Type:                    Projected (a volume that contains injected data from multiple sources)
		TokenExpirationSeconds:  3607
		ConfigMapName:           kube-root-ca.crt
		Optional:                false
		DownwardAPI:             true
	QoS Class:                   BestEffort
	Node-Selectors:              <none>
	Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
								node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
	Events:
	Type    Reason     Age   From               Message
	----    ------     ----  ----               -------
	Normal  Scheduled  2m2s  default-scheduler  Successfully assigned default/pingpong to pi-black
	Normal  Pulling    2m2s  kubelet            Pulling image "alpine"
	Normal  Pulled     2m    kubelet            Successfully pulled image "alpine" in 1.657s (1.657s including waiting). Image size: 4146781 bytes.
	Normal  Created    2m    kubelet            Created container: pingpong
	Normal  Started    2m    kubelet            Started container pingpong

  ```

]

(We should notice a bunch of control plane pods.)

---

## Listing running containers

- Containers are manipulated through *pods*

- A pod is a group of containers:

 - running together (on the same node)

 - sharing resources (RAM, CPU; but also network, volumes)

- List pods on our cluster:
  ```bash
	pi@pi-red:~ $ kubectl get pods
	NAME       READY   STATUS    RESTARTS   AGE
	pingpong   1/1     Running   0          3m17s
  ```

*Where are the pods that we saw just a moment earlier?!?*

---

## Namespaces

- Namespaces allow us to segregate resources

- List the namespaces on our cluster with one of these commands:
  ```bash
	pi@pi-red:~ $ kubectl get namespaces
	NAME              STATUS   AGE
	default           Active   25m
	kube-node-lease   Active   25m
	kube-public       Active   25m
	kube-system       Active   25m
	pi@pi-red:~ $ kubectl get namespace
	NAME              STATUS   AGE
	default           Active   26m
	kube-node-lease   Active   26m
	kube-public       Active   26m
	kube-system       Active   26m
	pi@pi-red:~ $ kubectl get ns
	NAME              STATUS   AGE
	default           Active   26m
	kube-node-lease   Active   26m
	kube-public       Active   26m
	kube-system       Active   26m
  ```

## Accessing namespaces

- By default, `kubectl` uses the `default` namespace

- We can see resources in all namespaces with `--all-namespaces`

- List the pods in all namespaces:
  ```bash
	pi@pi-red:~ $ kubectl get pods --all-namespaces
	NAMESPACE      NAME                             READY   STATUS    RESTARTS   AGE
	default        pingpong                         1/1     Running   0          76s
	kube-flannel   kube-flannel-ds-86tlq            1/1     Running   0          4m50s
	kube-flannel   kube-flannel-ds-kxlrn            1/1     Running   0          4m50s
	kube-system    coredns-674b8bbfcf-2t9wl         1/1     Running   0          4m5s
	kube-system    coredns-674b8bbfcf-gjtsz         1/1     Running   0          3m58s
	kube-system    etcd-pi-red                      1/1     Running   7          58m
	kube-system    kube-apiserver-pi-red            1/1     Running   7          58m
	kube-system    kube-controller-manager-pi-red   1/1     Running   4          58m
	kube-system    kube-proxy-5qfkn                 1/1     Running   0          58m
	kube-system    kube-proxy-8sdcn                 1/1     Running   0          58m
	kube-system    kube-scheduler-pi-red            1/1     Running   7          58m
  ```

- Since Kubernetes 1.14, we can also use `-A` as a shorter version:
  ```bash
	pi@pi-red:~ $ kubectl get pods -A
	NAMESPACE      NAME                             READY   STATUS    RESTARTS   AGE
	default        pingpong                         1/1     Running   0          5s
	kube-flannel   kube-flannel-ds-86tlq            1/1     Running   0          3m39s
	kube-flannel   kube-flannel-ds-kxlrn            1/1     Running   0          3m39s
	kube-system    coredns-674b8bbfcf-2t9wl         1/1     Running   0          2m54s
	kube-system    coredns-674b8bbfcf-gjtsz         1/1     Running   0          2m47s
	kube-system    etcd-pi-red                      1/1     Running   7          57m
	kube-system    kube-apiserver-pi-red            1/1     Running   7          57m
	kube-system    kube-controller-manager-pi-red   1/1     Running   4          57m
	kube-system    kube-proxy-5qfkn                 1/1     Running   0          57m
	kube-system    kube-proxy-8sdcn                 1/1     Running   0          56m
	kube-system    kube-scheduler-pi-red            1/1     Running   7          57m
  ```

*Here are our system pods!*

---

## What are all these control plane pods?

- `etcd` is our etcd server

- `kube-apiserver` is the API server

- `kube-controller-manager` and `kube-scheduler` are other control plane components

- `coredns` provides DNS-based service discovery ([replacing kube-dns as of 1.11](https://kubernetes.io/blog/2018/07/10/coredns-ga-for-kubernetes-cluster-dns/))

- `kube-proxy` is the (per-node) component managing port mappings and such

- `weave` is the (per-node) component managing the network overlay

- the `READY` column indicates the number of containers in each pod

---

## Scoping another namespace

- We can also look at a different namespace (other than `default`)

.lab[

- List only the pods in the `kube-system` namespace:
  ```bash
	pi@pi-red:~ $ kubectl get pods --namespace=kube-system
	NAME                             READY   STATUS              RESTARTS   AGE
	coredns-674b8bbfcf-b7zbz         0/1     ContainerCreating   0          29m
	coredns-674b8bbfcf-hk8mh         0/1     ContainerCreating   0          29m
	etcd-pi-red                      1/1     Running             7          29m
	kube-apiserver-pi-red            1/1     Running             7          29m
	kube-controller-manager-pi-red   1/1     Running             4          29m
	kube-proxy-5qfkn                 1/1     Running             0          29m
	kube-proxy-8sdcn                 1/1     Running             0          28m
	kube-scheduler-pi-red            1/1     Running             7          29m

	pi@pi-red:~ $ kubectl get pods -n kube-system
	NAME                             READY   STATUS              RESTARTS   AGE
	coredns-674b8bbfcf-b7zbz         0/1     ContainerCreating   0          29m
	coredns-674b8bbfcf-hk8mh         0/1     ContainerCreating   0          29m
	etcd-pi-red                      1/1     Running             7          29m
	kube-apiserver-pi-red            1/1     Running             7          29m
	kube-controller-manager-pi-red   1/1     Running             4          29m
	kube-proxy-5qfkn                 1/1     Running             0          29m
	kube-proxy-8sdcn                 1/1     Running             0          28m
	kube-scheduler-pi-red            1/1     Running             7          29m
  ```

## Namespaces and other `kubectl` commands

- We can use `-n`/`--namespace` with almost every `kubectl` command

- Example:

  - `kubectl create --namespace=X` to create something in namespace X

- We can use `-A`/`--all-namespaces` with most commands that manipulate multiple objects

- Examples:

  - `kubectl delete` can delete resources across multiple namespaces

```bash
pi@pi-red:~ $ kubectl delete pod pingpong
pod "pingpong" deleted
```

  - `kubectl label` can add/remove/update labels across multiple namespaces

  ```bash
	pi@pi-red:~ $ kubectl label pod pingpong status=pp
	pod/pingpong labeled
  ```
  Then you can see lable in describe pod

  ```
	Labels:         run=pingpong
					status=pp
  ```

## Services

- A *service* is a stable endpoint to connect to "something"

  (In the initial proposal, they were called "portals")

- List the services on our cluster with one of these commands:
  ```bash
	pi@pi-red:~ $ kubectl get services
	NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
	kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   33m

	pi@pi-red:~ $ kubectl get svc
	NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
	kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   34m
  ```

There is already one service on our cluster: the Kubernetes API itself.

## ClusterIP services

- A `ClusterIP` service is internal, available from the cluster only

- This is useful for introspection from within containers

- Try to connect to the API:
  ```bash
	~/development/kubernetes ❯ curl -k https://10.96.0.1                                                                                                                           frappe
	curl: (28) Failed to connect to 10.96.0.1 port 443 after 75003 ms: Couldn't connect to server
  ```

  - `-k` is used to skip certificate verification

  - Make sure to replace 10.96.0.1 with the CLUSTER-IP shown by `kubectl get svc`



The command above should either time out, or show an authentication error. Why?

---

## Time out

- Connections to ClusterIP services only work *from within the cluster*

- If we are outside the cluster, the `curl` command will probably time out

  (Because the IP address, e.g. 10.96.0.1, isn't routed properly outside the cluster)

- This is the case with most "real" Kubernetes clusters

- To try the connection from within the cluster, we can use [shpod](https://github.com/jpetazzo/shpod)

---

## Authentication error

This is what we should see when connecting from within the cluster:
```json
pi@pi-red:~ $ curl -k https://10.96.0.1
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```



## Explanations

- We can see `kind`, `apiVersion`, `metadata`

- These are typical of a Kubernetes API reply

- Because we *are* talking to the Kubernetes API

- The Kubernetes API tells us "Forbidden"

  (because it requires authentication)

- The Kubernetes API is reachable from within the cluster

  (many apps integrating with Kubernetes will use this)

---

## DNS integration

- Each service also gets a DNS record

- The Kubernetes DNS resolver is available *from within pods*

  (and sometimes, from within nodes, depending on configuration)

- Code running in pods can connect to services using their name

  (e.g. https://kubernetes/...)

# Running our first containers on Kubernetes

- First things first: we cannot run a container

- We are going to run a pod, and in that pod there will be a single container

- In that container in the pod, we are going to run a simple `ping` command

## Starting a simple pod with `kubectl run`

- `kubectl run` is convenient to start a single pod

- We need to specify at least a *name* and the image we want to use

- Optionally, we can specify the command to run in the pod

- Let's ping the address of `localhost`, the loopback interface:
  ```bash
	pi@pi-red:~ $ kubectl run pingpong --image alpine ping 127.0.0.1
	pod/pingpong created
  ```
The output tells us that a Pod was created:

## Viewing container output

- Let's use the `kubectl logs` command

- It takes a Pod name as argument

- Unless specified otherwise, it will only show logs of the first container in the pod

  (Good thing there's only one in ours!)

- View the result of our `ping` command:
  ```bash
	pi@pi-red:~ $ kubectl logs pingpong
	PING 127.0.0.1 (127.0.0.1): 56 data bytes
	64 bytes from 127.0.0.1: seq=0 ttl=64 time=0.115 ms
	64 bytes from 127.0.0.1: seq=1 ttl=64 time=0.097 ms
	64 bytes from 127.0.0.1: seq=2 ttl=64 time=0.089 ms
	64 bytes from 127.0.0.1: seq=3 ttl=64 time=0.178 ms
  ```

---

## Streaming logs in real time

- Just like `docker logs`, `kubectl logs` supports convenient options:

  - `-f`/`--follow` to stream logs in real time (à la `tail -f`)

  - `--tail` to indicate how many lines you want to see (from the end)
  ```bash
	pi@pi-red:~ $ kubectl logs pingpong --tail 5
	64 bytes from 127.0.0.1: seq=299 ttl=64 time=0.205 ms
	64 bytes from 127.0.0.1: seq=300 ttl=64 time=0.182 ms
	64 bytes from 127.0.0.1: seq=301 ttl=64 time=0.188 ms
	64 bytes from 127.0.0.1: seq=302 ttl=64 time=0.174 ms
	64 bytes from 127.0.0.1: seq=303 ttl=64 time=0.182 ms
  ```

  - `--since` to get logs only after a given timestamp


- View the latest logs of our `ping` command:
  ```bash
  kubectl logs pingpong --tail 1 --follow
  ```

## Scaling our application

- `kubectl` gives us a simple command to scale a workload:

  `kubectl scale TYPE NAME --replicas=HOWMANY`

- Let's try it on our Pod, so that we have more Pods!

.lab[

- Try to scale the Pod:
  ```bash
  kubectl scale pod pingpong --replicas=3
  ```

]

🤔 We get the following error, what does that mean?

```
Error from server (NotFound): the server could not find the requested resource
```

---

## Scaling a Pod

- We cannot "scale a Pod"

  (that's not completely true; we could give it more CPU/RAM)

- If we want more Pods, we need to create more Pods

  (i.e. execute `kubectl run` multiple times)

- There must be a better way!

  (spoiler alert: yes, there is a better way!)


## `NotFound`

- What's the meaning of that error?
  ```
  Error from server (NotFound): the server could not find the requested resource
  ```

- When we execute `kubectl scale THAT-RESOURCE --replicas=THAT-MANY`,
  <br/>
  it is like telling Kubernetes:

  *go to THAT-RESOURCE and set the scaling button to position THAT-MANY*

- Pods do not have a "scaling button"

- Try to execute the `kubectl scale pod` command with `-v6`

```bash
pi@pi-red:~ $ kubectl scale pod pingpong --replicas=3 -v6
I0710 22:17:18.224173   28248 loader.go:402] Config loaded from file:  /home/pi/.kube/config
I0710 22:17:18.225725   28248 envvar.go:172] "Feature gate default state" feature="InformerResourceVersion" enabled=false
I0710 22:17:18.225826   28248 envvar.go:172] "Feature gate default state" feature="InOrderInformers" enabled=true
I0710 22:17:18.225866   28248 envvar.go:172] "Feature gate default state" feature="WatchListClient" enabled=false
I0710 22:17:18.225903   28248 envvar.go:172] "Feature gate default state" feature="ClientsAllowCBOR" enabled=false
I0710 22:17:18.225934   28248 envvar.go:172] "Feature gate default state" feature="ClientsPreferCBOR" enabled=false
I0710 22:17:18.267011   28248 round_trippers.go:632] "Response" verb="GET" url="https://192.168.0.152:6443/api/v1/namespaces/default/pods/pingpong" status="200 OK" milliseconds=30
I0710 22:17:18.270831   28248 round_trippers.go:632] "Response" verb="PATCH" url="https://192.168.0.152:6443/api/v1/namespaces/default/pods/pingpong/scale" status="404 Not Found" milliseconds=2
I0710 22:17:18.271588   28248 helpers.go:246] server response object: %s[{
  "metadata": {},
  "status": "Failure",
  "message": "the server could not find the requested resource",
  "reason": "NotFound",
  "details": {},
  "code": 404
}]
Error from server (NotFound): the server could not find the requested resource
```

- We see a `PATCH` request to `/scale`: that's the "scaling button"

  (technically it's called a *subresource* of the Pod)

---

## Creating more pods

- We are going to create a ReplicaSet

  (= set of replicas = set of identical pods)

- In fact, we will create a Deployment, which itself will create a ReplicaSet

- Why so many layers? We'll explain that shortly, don't worry!

## Creating a Deployment running `ping`

- Let's create a Deployment instead of a single Pod

- Create the Deployment:
  ```bash
	pi@pi-red:~ $ cat pingpong.yaml 
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	name: pingpong
	spec:
	replicas: 1
	selector:
		matchLabels:
		app: pingpong
	template:
		metadata:
		labels:
			app: pingpong
		spec:
		containers:
		- name: pingpong
			image: alpine
			command: ["ping", "127.0.0.1"]

	pi@pi-red:~ $ kubectl apply -f pingpong.yaml
	deployment.apps/pingpong created
	pi@pi-red:~ $ kubectl get deployments
	NAME       READY   UP-TO-DATE   AVAILABLE   AGE
	pingpong   1/1     1            1           38s
	
	pi@pi-red:~ $ kubectl scale deployment pingpong --replicas=3
	deployment.apps/pingpong scaled
  ```

## What has been created?

- Check the resources that were created:
  ```bash
  kubectl get all
  ```

Note: `kubectl get all` is a lie. It doesn't show everything.

(But it shows a lot of "usual suspects", i.e. commonly used resources.)

---

## There's a lot going on here!

```bash
pi@pi-red:~ $ kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/pingpong-5859ffb968-9fx8c   1/1     Running   0          2m16s
pod/pingpong-5859ffb968-jkr89   1/1     Running   0          17s
pod/pingpong-5859ffb968-v266p   1/1     Running   0          17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   7m35s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/pingpong   3/3     3            3           2m16s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/pingpong-5859ffb968   3         3         3       2m16s
```
```bash
pi@pi-red:~ $ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
pingpong                    1/1     Running   0          11m
pingpong-5949c46847-rvk4b   1/1     Running   0          13s
pi@pi-red:~ $ kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
pingpong   1/1     1            1           27s
pi@pi-red:~ $ kubectl get replicasets
NAME                  DESIRED   CURRENT   READY   AGE
pingpong-5949c46847   1         1         1       35s
```

Our new Pod is not named `pingpong`, but `pingpong-xxxxxxxxxxx-yyyyy`.

We have a Deployment named `pingpong`, and an extra ReplicaSet, too. What's going on?

---

## From Deployment to Pod

We have the following resources:

- `deployment.apps/pingpong`

  This is the Deployment that we just created.

- `replicaset.apps/pingpong-xxxxxxxxxx`

  This is a Replica Set created by this Deployment.

- `pod/pingpong-xxxxxxxxxx-yyyyy`

  This is a *pod* created by the Replica Set.

Let's explain what these things are.

---

## Pod

- Can have one or multiple containers

- Runs on a single node

  (Pod cannot "straddle" multiple nodes)

- Pods cannot be moved

  (e.g. in case of node outage)

- Pods cannot be scaled horizontally

  (except by manually creating more Pods)


## Pod details

- A Pod is not a process; it's an environment for containers

  - it cannot be "restarted"

  - it cannot "crash"

- The containers in a Pod can crash

- They may or may not get restarted

  (depending on Pod's restart policy)

- If all containers exit successfully, the Pod ends in "Succeeded" phase

- If some containers fail and don't get restarted, the Pod ends in "Failed" phase

---

## Replica Set

- Set of identical (replicated) Pods

- Defined by a pod template + number of desired replicas

- If there are not enough Pods, the Replica Set creates more

  (e.g. in case of node outage; or simply when scaling up)

- If there are too many Pods, the Replica Set deletes some

  (e.g. if a node was disconnected and comes back; or when scaling down)

- We can scale up/down a Replica Set

  - we update the manifest of the Replica Set

  - as a consequence, the Replica Set controller creates/deletes Pods

---

## Deployment

- Replica Sets control *identical* Pods

- Deployments are used to roll out different Pods

  (different image, command, environment variables, ...)

- When we update a Deployment with a new Pod definition:

  - a new Replica Set is created with the new Pod definition

  - that new Replica Set is progressively scaled up

  - meanwhile, the old Replica Set(s) is(are) scaled down

- This is a *rolling update*, minimizing application downtime

- When we scale up/down a Deployment, it scales up/down its Replica Set

---

## Can we scale now?

- Let's try `kubectl scale` again, but on the Deployment!

.lab[

- Scale our `pingpong` deployment:
  ```bash
	pi@pi-red:~ $ kubectl scale deployment pingpong --replicas 3
	deployment.apps/pingpong scaled
	pi@pi-red:~ $ kubectl get replicasets
	NAME                  DESIRED   CURRENT   READY   AGE
	pingpong-5949c46847   3         3         1       4m15s
	pi@pi-red:~ $ kubectl get pods
	NAME                        READY   STATUS    RESTARTS   AGE
	pingpong                    1/1     Running   0          15m
	pingpong-5949c46847-77f7p   1/1     Running   0          8s
	pingpong-5949c46847-7m89k   1/1     Running   0          8s
	pingpong-5949c46847-rvk4b   1/1     Running   0          4m19s
  ```

- Note that we could also write it like this:
  ```bash
  kubectl scale deployment/pingpong --replicas 3
  ```

## Scaling a Replica Set

- What if we scale the Replica Set instead of the Deployment?

- The Deployment would notice it right away and scale back to the initial level

- The Replica Set makes sure that we have the right numbers of Pods

- The Deployment makes sure that the Replica Set has the right size

  (conceptually, it delegates the management of the Pods to the Replica Set)

- This might seem weird (why this extra layer?) but will soon make sense

  (when we will look at how rolling updates work!)

---

## Checking Deployment logs

- `kubectl logs` needs a Pod name

- But it can also work with a *type/name*

  (e.g. `deployment/pingpong`)

- View the result of our `ping` command:
  ```bash
	pi@pi-red:~ $ kubectl logs deploy/pingpong --tail 2
	Found 3 pods, using pod/pingpong-5949c46847-rvk4b
	64 bytes from 127.0.0.1: seq=321 ttl=64 time=0.102 ms
	64 bytes from 127.0.0.1: seq=322 ttl=64 time=0.127 ms
  ```
- It shows us the logs of the first Pod of the Deployment

- We'll see later how to get the logs of *all* the Pods!


## Resilience

- The *deployment* `pingpong` watches its *replica set*

- The *replica set* ensures that the right number of *pods* are running

- What happens if pods disappear?

- In a separate window, watch the list of pods:
  ```bash
  watch kubectl get pods
	Every 2.0s: kubectl get pods                          pi-red: Thu Jul 10 22:27:29 2025

	NAME                        READY   STATUS    RESTARTS   AGE
	pingpong                    1/1     Running   0          19m
	pingpong-5949c46847-7m89k   1/1     Running   0          4m25s
	pingpong-5949c46847-g7cmd   1/1     Running   0          74s
	pingpong-5949c46847-rvk4b   1/1     Running   0          8m36s
  ```

- Destroy the pod currently shown by `kubectl logs`:
  ```
	pi@pi-red:~ $ kubectl delete pod pingpong-5949c46847-rvk4b
	pod "pingpong-5949c46847-rvk4b" deleted

	Every 2.0s: kubectl get pods                          pi-red: Thu Jul 10 22:28:26 2025

	NAME                        READY   STATUS        RESTARTS   AGE
	pingpong                    1/1     Running       0          20m
	pingpong-5949c46847-7m89k   1/1     Running       0          5m22s
	pingpong-5949c46847-fccph   1/1     Running       0          4s
	pingpong-5949c46847-g7cmd   1/1     Running       0          2m11s
	pingpong-5949c46847-rvk4b   1/1     Terminating   0          9m33s
  ```

## What happened?

- `kubectl delete pod` terminates the pod gracefully

  (sending it the TERM signal and waiting for it to shutdown)

- As soon as the pod is in "Terminating" state, the Replica Set replaces it

- But we can still see the output of the "Terminating" pod in `kubectl logs`

- Until 30 seconds later, when the grace period expires

- The pod is then killed, and `kubectl logs` exits

---

## Deleting a standalone Pod

- What happens if we delete a standalone Pod?
 
  (like the first `pingpong` Pod that we created)

- Delete the Pod:
  ```bash
  kubectl delete pod pingpong
  ```

- No replacement Pod gets created because there is no *controller* watching it

- That's why we will rarely use standalone Pods in practice

  (except for e.g. punctual debugging or executing a short supervised task)

# Executing batch jobs

- Deployments are great for stateless web apps

  (as well as workers that keep running forever)

- Pods are great for one-off execution that we don't care about

  (because they don't get automatically restarted if something goes wrong)

- Jobs are great for "long" background work

  ("long" being at least minutes or hours)

- CronJobs are great to schedule Jobs at regular intervals

  (just like the classic UNIX `cron` daemon with its `crontab` files)

---

## Creating a Job

- A Job will create a Pod

- If the Pod fails, the Job will create another one

- The Job will keep trying until:

  - either a Pod succeeds,

  - or we hit the *backoff limit* of the Job (default=6)

- Create a Job that has a 50% chance of success:
  ```bash
	pi@pi-red:~ $ kubectl create job flipcoin --image=alpine -- sh -c 'exit $(($RANDOM%2)'
	job.batch/flipcoin created 
  ```


---

## Our Job in action

- Our Job will create a Pod named `flipcoin-xxxxx`

- If the Pod succeeds, the Job stops

- If the Pod fails, the Job creates another Pod

- Check the status of the Pod(s) created by the Job:
  ```bash
	pi@pi-red:~ $ kubectl get pods --selector=job-name=flipcoin
	NAME             READY   STATUS      RESTARTS   AGE
	flipcoin-gnnzw   0/1     Completed   0          29s
  ```


## More advanced jobs

- We can specify a number of "completions" (default=1)

- This indicates how many times the Job must be executed

- We can specify the "parallelism" (default=1)

- This indicates how many Pods should be running in parallel

- These options cannot be specified with `kubectl create job`

  (we have to write our own YAML manifest to use them)

---

## Scheduling periodic background work

- A Cron Job is a Job that will be executed at specific intervals

  (the name comes from the traditional cronjobs executed by the UNIX crond)

- It requires a *schedule*, represented as five space-separated fields:

  - minute [0,59]
  - hour [0,23]
  - day of the month [1,31]
  - month of the year [1,12]
  - day of the week ([0,6] with 0=Sunday)

- `*` means "all valid values"; `/N` means "every N"

- Example: `*/3 * * * *` means "every three minutes"

- The website https://crontab.guru/ can help to create cron schedules!

---

## Creating a Cron Job

- Let's create a simple job to be executed every three minutes

- Careful: make sure that the job terminates!

  (The Cron Job will not hold if a previous job is still running)

.lab[

- Create the Cron Job:
  ```bash
	pi@pi-red:~ $ kubectl create cronjob every3mins --schedule="*/3 * * * *" \
				--image=alpine -- sleep 10
	cronjob.batch/every3mins created
  ```

- Check the resource that was created:
  ```bash
	pi@pi-red:~ $ kubectl get cronjobs
	NAME         SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
	every3mins   */3 * * * *   <none>     False     0        <none>          73s
	pi@pi-red:~ $ kubectl get cronjobs
	NAME         SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
	every3mins   */3 * * * *   <none>     False     0        81s             3m1s
	pi@pi-red:~ $ kubectl get pods -A
	NAMESPACE      NAME                             READY   STATUS      RESTARTS   AGE
	default        every3mins-29202843-bfwzk        0/1     Completed   0          85s
	default        flipcoin-gnnzw                   0/1     Completed   0          5m51s
	default        pingpong                         1/1     Running     0          6m31s
	kube-flannel   kube-flannel-ds-5mww5            1/1     Running     0          8m58s
	kube-flannel   kube-flannel-ds-sfpls            1/1     Running     0          8m58s
	kube-system    coredns-674b8bbfcf-9wgz2         1/1     Running     0          12m
	kube-system    coredns-674b8bbfcf-hq7fr         1/1     Running     0          12m
	kube-system    etcd-pi-red                      1/1     Running     9          12m
	kube-system    kube-apiserver-pi-red            1/1     Running     9          12m
	kube-system    kube-controller-manager-pi-red   1/1     Running     6          12m
	kube-system    kube-proxy-fjhl4                 1/1     Running     0          12m
	kube-system    kube-proxy-fsbfz                 1/1     Running     0          10m
	kube-system    kube-scheduler-pi-red            1/1     Running     9          12m
  ```

]

---

## Cron Jobs in action

- At the specified schedule, the Cron Job will create a Job

- The Job will create a Pod

- The Job will make sure that the Pod completes

  (re-creating another one if it fails, for instance if its node fails)

- Check the Jobs that are created:
  ```bash
	pi@pi-red:~ $ kubectl get jobs
	NAME                  STATUS     COMPLETIONS   DURATION   AGE
	every3mins-29202843   Complete   1/1           15s        3m38s
	every3mins-29202846   Complete   1/1           15s        38s
	flipcoin              Complete   1/1           6s         8m4s
  ```

(It will take a few minutes before the first job is scheduled.)

## Setting a time limit

- It is possible to set a time limit (or deadline) for a job

- This is done with the field `spec.activeDeadlineSeconds`

  (by default, it is unlimited)

- When the job is older than this time limit, all its pods are terminated

- Note that there can also be a `spec.activeDeadlineSeconds` field in pods!

- They can be set independently, and have different effects:

  - the deadline of the job will stop the entire job

  - the deadline of the pod will only stop an individual pod


# Labels and annotations

- Most Kubernetes resources can have *labels* and *annotations*

- Both labels and annotations are arbitrary strings

  (with some limitations that we'll explain in a minute)

- Both labels and annotations can be added, removed, changed, dynamically

- This can be done with:

  - the `kubectl edit` command

  - the `kubectl label` and `kubectl annotate`

  - ... many other ways! (`kubectl apply -f`, `kubectl patch`, ...)

---

## Viewing labels and annotations

- Let's see what we get when we create a Deployment

- Create a Deployment:
  ```bash
	pi@pi-red:~ $ kubectl create deployment clock --image=jpetazzo/clock
	deployment.apps/clock created
  ```

- Look at its annotations and labels:
  ```bash
	pi@pi-red:~ $ kubectl describe deployment clock
	Name:                   clock
	Namespace:              default
	CreationTimestamp:      Mon, 14 Jul 2025 21:11:22 +0530
	Labels:                 app=clock
	Annotations:            deployment.kubernetes.io/revision: 1
	Selector:               app=clock
	Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
	StrategyType:           RollingUpdate
	MinReadySeconds:        0
	RollingUpdateStrategy:  25% max unavailable, 25% max surge
	Pod Template:
	Labels:  app=clock
	Containers:
	clock:
		Image:         jpetazzo/clock
		Port:          <none>
		Host Port:     <none>
		Environment:   <none>
		Mounts:        <none>
	Volumes:         <none>
	Node-Selectors:  <none>
	Tolerations:     <none>
	Conditions:
	Type           Status  Reason
	----           ------  ------
	Available      True    MinimumReplicasAvailable
	Progressing    True    NewReplicaSetAvailable
	OldReplicaSets:  <none>
	NewReplicaSet:   clock-6dbf954cc (1/1 replicas created)
	Events:
	Type    Reason             Age   From                   Message
	----    ------             ----  ----                   -------
	Normal  ScalingReplicaSet  27s   deployment-controller  Scaled up replica set clock-6dbf954cc from 0 to 1
  ```

## Labels and annotations for our Deployment

- We see one label:
  ```
  Labels: app=clock
  ```

- This is added by `kubectl create deployment`

- And one annotation:
  ```
  Annotations: deployment.kubernetes.io/revision: 1
  ```

- This is to keep track of successive versions when doing rolling updates

---

## And for the related Pod?

- Let's look up the Pod that was created and check it too

.lab[

- Find the name of the Pod:
  ```bash
	pi@pi-red:~ $ kubectl get pods
	NAME                    READY   STATUS    RESTARTS   AGE
	clock-6dbf954cc-mzp9x   1/1     Running   0          2m36s
  ```

- Display its information:
  ```bash
	pi@pi-red:~ $ kubectl describe pod clock-6dbf954cc-mzp9x
	Name:             clock-6dbf954cc-mzp9x
	Namespace:        default
	Priority:         0
	Service Account:  default
	Node:             pi-black/192.168.0.101
	Start Time:       Mon, 14 Jul 2025 21:11:22 +0530
	Labels:           app=clock
			    pod-template-hash=6dbf954cc
	Annotations:      <none>
	Status:           Running
	IP:               10.244.1.2
	IPs:
	IP:           10.244.1.2
	Controlled By:  ReplicaSet/clock-6dbf954cc
	Containers:
	clock:
		Container ID:   containerd://750389ffb0bdba94d324c653d0f9bd1722993efab4441cc8d9b66de211a06bbe
		Image:          jpetazzo/clock
		Image ID:       docker.io/jpetazzo/clock@sha256:dc06bbc3744f7200404bff0bbb2516925e7adea115e07de9da8b36bf15fe3dd3
		Port:           <none>
		Host Port:      <none>
		State:          Running
		Started:      Mon, 14 Jul 2025 21:11:31 +0530
		Ready:          True
		Restart Count:  0
		Environment:    <none>
		Mounts:
		/var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-495h9 (ro)
	Conditions:
	Type                        Status
	PodReadyToStartContainers   True 
	Initialized                 True 
	Ready                       True 
	ContainersReady             True 
	PodScheduled                True 
	Volumes:
	kube-api-access-495h9:
		Type:                    Projected (a volume that contains injected data from multiple sources)
		TokenExpirationSeconds:  3607
		ConfigMapName:           kube-root-ca.crt
		Optional:                false
		DownwardAPI:             true
	QoS Class:                   BestEffort
	Node-Selectors:              <none>
	Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
								node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
	Events:
	Type    Reason     Age    From               Message
	----    ------     ----   ----               -------
	Normal  Scheduled  2m59s  default-scheduler  Successfully assigned default/clock-6dbf954cc-mzp9x to pi-black
	Normal  Pulling    2m58s  kubelet            Pulling image "jpetazzo/clock"
	Normal  Pulled     2m50s  kubelet            Successfully pulled image "jpetazzo/clock" in 7.857s (7.857s including waiting). Image size: 819900 bytes.
	Normal  Created    2m50s  kubelet            Created container: clock
	Normal  Started    2m50s  kubelet            Started container clock
  ```

---

## Labels and annotations for our Pod

- We see two labels:
  ```
    Labels: app=clock
            pod-template-hash=xxxxxxxxxx
  ```

- `app=clock` comes from `kubectl create deployment` too

- `pod-template-hash` was assigned by the Replica Set

  (when we will do rolling updates, each set of Pods will have a different hash)

- There are no annotations:
  ```
  Annotations: <none>
  ```

---

## Selectors

- A *selector* is an expression matching labels

- It will restrict a command to the objects matching *at least* all these labels

- List all the pods with at least `app=clock`:
  ```bash
	pi@pi-red:~ $ kubectl get pods --selector=app=clock
	NAME                    READY   STATUS    RESTARTS   AGE
	clock-6dbf954cc-mzp9x   1/1     Running   0          5m44s
  ```

- List all the pods with a label `app`, regardless of its value:
  ```bash
	pi@pi-red:~ $ kubectl get pods --selector=app
	NAME                    READY   STATUS    RESTARTS   AGE
	clock-6dbf954cc-mzp9x   1/1     Running   0          6m1s
  ```

---

## Settings labels and annotations

- The easiest method is to use `kubectl label` and `kubectl annotate`

- Set a label on the `clock` Deployment:
  ```bash
	pi@pi-red:~ $ kubectl label deployment clock color=blue
	deployment.apps/clock labeled
  ```

- Check it out:
  ```bash
	pi@pi-red:~ $ kubectl describe deployment clock
	Name:                   clock
	Namespace:              default
	CreationTimestamp:      Mon, 14 Jul 2025 21:11:22 +0530
	Labels:                 app=clock
							color=blue
	Annotations:            deployment.kubernetes.io/revision: 1
	Selector:               app=clock
	Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
	StrategyType:           RollingUpdate
	MinReadySeconds:        0
	RollingUpdateStrategy:  25% max unavailable, 25% max surge
	Pod Template:
	Labels:  app=clock
	Containers:
	clock:
		Image:         jpetazzo/clock
		Port:          <none>
		Host Port:     <none>
		Environment:   <none>
		Mounts:        <none>
	Volumes:         <none>
	Node-Selectors:  <none>
	Tolerations:     <none>
	Conditions:
	Type           Status  Reason
	----           ------  ------
	Available      True    MinimumReplicasAvailable
	Progressing    True    NewReplicaSetAvailable
	OldReplicaSets:  <none>
	NewReplicaSet:   clock-6dbf954cc (1/1 replicas created)
	Events:
	Type    Reason             Age   From                   Message
	----    ------             ----  ----                   -------
	Normal  ScalingReplicaSet  7m6s  deployment-controller  Scaled up replica set clock-6dbf954cc from 0 to 1
  ```

---

## Other ways to view labels

- `kubectl get` gives us a couple of useful flags to check labels

- `kubectl get --show-labels` shows all labels

- `kubectl get -L xyz` shows the value of label `xyz`

- List all the labels that we have on pods:
  ```bash
	pi@pi-red:~ $ kubectl get pods
	NAME                    READY   STATUS    RESTARTS   AGE
	clock-6dbf954cc-mzp9x   1/1     Running   0          8m30s
	pi@pi-red:~ $ kubectl label pod clock-6dbf954cc-mzp9x is_new=No
	pod/clock-6dbf954cc-mzp9x labeled

	pi@pi-red:~ $ kubectl get pods --show-labels
	NAME                    READY   STATUS    RESTARTS   AGE     LABELS
	clock-6dbf954cc-mzp9x   1/1     Running   0          9m47s   app=clock,is_new=No,pod-template-hash=6dbf954cc

	pi@pi-red:~ $ kubectl get pods -L is_new
	NAME                    READY   STATUS    RESTARTS   AGE   IS_NEW
	clock-6dbf954cc-mzp9x   1/1     Running   0          11m   No
  ```

- List the value of label `app` on these pods:
  ```bash
	pi@pi-red:~ $ kubectl get pods -L app
	NAME                    READY   STATUS    RESTARTS   AGE   APP
	clock-6dbf954cc-mzp9x   1/1     Running   0          11m   clock
  ```

## More on selectors

- If a selector has multiple labels, it means "match at least these labels"

  Example: `--selector=app=frontend,release=prod`

- `--selector` can be abbreviated as `-l` (for **l**abels)

  We can also use negative selectors

  Example: `--selector=app!=clock`

- Selectors can be used with most `kubectl` commands

  Examples: `kubectl delete`, `kubectl label`, ...

---

## Other ways to view labels

- We can use the `--show-labels` flag with `kubectl get`

.lab[

- Show labels for a bunch of objects:
  ```bash
	pi@pi-red:~ $ kubectl get --show-labels po,rs,deploy,svc,no
	NAME                        READY   STATUS    RESTARTS   AGE   LABELS
	pod/clock-6dbf954cc-mzp9x   1/1     Running   0          12m   app=clock,is_new=No,pod-template-hash=6dbf954cc

	NAME                              DESIRED   CURRENT   READY   AGE   LABELS
	replicaset.apps/clock-6dbf954cc   1         1         1       12m   app=clock,pod-template-hash=6dbf954cc

	NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
	deployment.apps/clock   1/1     1            1           12m   app=clock,color=blue

	NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   LABELS
	service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   23m   component=apiserver,provider=kubernetes

	NAME            STATUS   ROLES           AGE   VERSION   LABELS
	node/pi-black   Ready    <none>          20m   v1.33.2   beta.kubernetes.io/arch=arm64,beta.kubernetes.io/os=linux,kubernetes.io/arch=arm64,kubernetes.io/hostname=pi-black,kubernetes.io/os=linux
	node/pi-red     Ready    control-plane   23m   v1.33.2   beta.kubernetes.io/arch=arm64,beta.kubernetes.io/os=linux,kubernetes.io/arch=arm64,kubernetes.io/hostname=pi-red,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node.kubernetes.io/exclude-from-external-load-balancers=

	pi@pi-red:~ $ kubectl get --show-labels po
	NAME                    READY   STATUS    RESTARTS   AGE   LABELS
	clock-6dbf954cc-mzp9x   1/1     Running   0          13m   app=clock,is_new=No,pod-template-hash=6dbf954cc
  ```

## Differences between labels and annotations

- The *key* for both labels and annotations:

  - must start and end with a letter or digit

  - can also have `.` `-` `_` (but not in first or last position)

  - can be up to 63 characters, or 253 + `/` + 63

- Label *values* are up to 63 characters, with the same restrictions

- Annotations *values* can have arbitrary characters (yes, even binary)

- Maximum length isn't defined

  (dozens of kilobytes is fine, hundreds maybe not so much)

# Revisiting `kubectl logs`

- In this section, we assume that we have a Deployment with multiple Pods

  (e.g. `pingpong` that we scaled to at least 3 pods)

- We will highlights some of the limitations of `kubectl logs`

---

## Streaming logs of multiple pods

- By default, `kubectl logs` shows us the output of a single Pod

- Try to check the output of the Pods related to a Deployment:
  ```bash
	pi@pi-red:~ $ kubectl logs deploy/pingpong --tail 1 --follow
	Found 3 pods, using pod/pingpong-5859ffb968-9fx8c
	64 bytes from 127.0.0.1: seq=288 ttl=64 time=0.187 ms
	64 bytes from 127.0.0.1: seq=289 ttl=64 time=0.167 ms
	64 bytes from 127.0.0.1: seq=290 ttl=64 time=0.154 ms
	64 bytes from 127.0.0.1: seq=291 ttl=64 time=0.163 ms
  ```

`kubectl logs` only shows us the logs of one of the Pods.

---

## Viewing logs of multiple pods

- When we specify a deployment name, only one single pod's logs are shown

- We can view the logs of multiple pods by specifying a *selector*

- If we check the pods created by the deployment, they all have the label `app=pingpong`

  (this is just a default label that gets added when using `kubectl create deployment`)

- View the last line of log from all pods with the `app=pingpong` label:
  ```bash
	pi@pi-red:~ $ kubectl logs -l app=pingpong --tail 1
	64 bytes from 127.0.0.1: seq=356 ttl=64 time=0.234 ms
	64 bytes from 127.0.0.1: seq=235 ttl=64 time=0.233 ms
	64 bytes from 127.0.0.1: seq=237 ttl=64 time=0.167 ms
  ```
- See the sequence from which you can identify these logs are from different pods while if we fetch logs from one pod then we will get those logs sequentially

---

## Streaming logs of multiple pods

- Can we stream the logs of all our `pingpong` pods?

.lab[

- Combine `-l` and `-f` flags:
  ```bash
	pi@pi-red:~ $ kubectl logs -l app=pingpong --tail 1 -f
	64 bytes from 127.0.0.1: seq=496 ttl=64 time=0.189 ms
	64 bytes from 127.0.0.1: seq=375 ttl=64 time=0.102 ms
	64 bytes from 127.0.0.1: seq=377 ttl=64 time=0.097 ms
	64 bytes from 127.0.0.1: seq=497 ttl=64 time=0.098 ms
	64 bytes from 127.0.0.1: seq=376 ttl=64 time=0.104 ms
	64 bytes from 127.0.0.1: seq=378 ttl=64 time=0.101 ms
	64 bytes from 127.0.0.1: seq=498 ttl=64 time=0.105 ms
	64 bytes from 127.0.0.1: seq=377 ttl=64 time=0.099 ms
	64 bytes from 127.0.0.1: seq=379 ttl=64 time=0.109 ms
	64 bytes from 127.0.0.1: seq=499 ttl=64 time=0.107 ms
	64 bytes from 127.0.0.1: seq=378 ttl=64 time=0.105 ms
	64 bytes from 127.0.0.1: seq=380 ttl=64 time=0.096 ms
	64 bytes from 127.0.0.1: seq=500 ttl=64 time=0.097 ms
	64 bytes from 127.0.0.1: seq=379 ttl=64 time=0.101 ms
	64 bytes from 127.0.0.1: seq=381 ttl=64 time=0.104 ms
	64 bytes from 127.0.0.1: seq=501 ttl=64 time=0.104 ms
	64 bytes from 127.0.0.1: seq=380 ttl=64 time=0.207 ms
  ```

*Note: combining `-l` and `-f` is only possible since Kubernetes 1.14!*

*Let's try to understand why ...*


## Streaming logs of many pods

- Let's see what happens if we try to stream the logs for more than 5 pods

- Scale up our deployment:
  ```bash
  kubectl scale deployment pingpong --replicas=8
  ```

- Stream the logs:
  ```bash
	pi@pi-red:~ $ kubectl logs -l app=pingpong --tail 1 -f
	error: you are attempting to follow 8 log streams, but maximum allowed concurrency is 5, use --max-log-requests to increase the limit
  ```

We see a message like the following one:
```
error: you are attempting to follow 8 log streams,
but maximum allowed concurency is 5,
use --max-log-requests to increase the limit
```

## Why can't we stream the logs of many pods?

- `kubectl` opens one connection to the API server per pod

- For each pod, the API server opens one extra connection to the corresponding kubelet

- If there are 1000 pods in our deployment, that's 1000 inbound + 1000 outbound connections on the API server

- This could easily put a lot of stress on the API server

- Prior Kubernetes 1.14, it was decided to *not* allow multiple connections

- From Kubernetes 1.14, it is allowed, but limited to 5 connections

  (this can be changed with `--max-log-requests`)

- For more details about the rationale, see
  [PR #67573](https://github.com/kubernetes/kubernetes/pull/67573)

## Shortcomings of `kubectl logs`

- We don't see which pod sent which log line

- If pods are restarted / replaced, the log stream stops

- If new pods are added, we don't see their logs

- To stream the logs of multiple pods, we need to write a selector

- There are external tools to address these shortcomings

  (e.g.: [Stern](https://github.com/stern/stern))


## `kubectl logs -l ... --tail N`

- If we run this with Kubernetes 1.12, the last command shows multiple lines

- This is a regression when `--tail` is used together with `-l`/`--selector`

- It always shows the last 10 lines of output for each container

  (instead of the number of lines specified on the command line)

- The problem was fixed in Kubernetes 1.13

*See [#70554](https://github.com/kubernetes/kubernetes/issues/70554) for details.*

# Accessing logs from the CLI

- The `kubectl logs` command has limitations:

  - it cannot stream logs from multiple pods at a time

  - when showing logs from multiple pods, it mixes them all together

- We are going to see how to do it better

---

## Doing it manually

- We *could* (if we were so inclined) write a program or script that would:

  - take a selector as an argument

  - enumerate all pods matching that selector (with `kubectl get -l ...`)

  - fork one `kubectl logs --follow ...` command per container

  - annotate the logs (the output of each `kubectl logs ...` process) with their origin

  - preserve ordering by using `kubectl logs --timestamps ...` and merge the output

--

- We *could* do it, but thankfully, others did it for us already!

---

## Stern

[Stern](https://github.com/stern/stern) is an open source project
originally by [Wercker](http://www.wercker.com/).

From the README:

*Stern allows you to tail multiple pods on Kubernetes and multiple containers within the pod. Each result is color coded for quicker debugging.*

*The query is a regular expression so the pod name can easily be filtered and you don't need to specify the exact id (for instance omitting the deployment id). If a pod is deleted it gets removed from tail and if a new pod is added it automatically gets tailed.*

# Declarative vs imperative

- Our container orchestrator puts a very strong emphasis on being *declarative*

- Declarative:

  *I would like a cup of tea.*

- Imperative:

  *Boil some water. Pour it in a teapot. Add tea leaves. Steep for a while. Serve in a cup.*

- Declarative seems simpler at first ... 

- ... As long as you know how to brew tea

---

## Declarative vs imperative

- What declarative would really be:

  *I want a cup of tea, obtained by pouring an infusion¹ of tea leaves in a cup.*

--

  *¹An infusion is obtained by letting the object steep a few minutes in hot² water.*

--

  *²Hot liquid is obtained by pouring it in an appropriate container³ and setting it on a stove.*

--

  *³Ah, finally, containers! Something we know about. Let's get to work, shall we?*

## Declarative vs imperative

- Imperative systems:

  - simpler

  - if a task is interrupted, we have to restart from scratch

- Declarative systems:

  - if a task is interrupted (or if we show up to the party half-way through),
    we can figure out what's missing and do only what's necessary

  - we need to be able to *observe* the system

  - ... and compute a "diff" between *what we have* and *what we want*

## Declarative vs imperative in Kubernetes

- With Kubernetes, we cannot say: "run this container"

- All we can do is write a *spec* and push it to the API server

  (by creating a resource like e.g. a Pod or a Deployment)

- The API server will validate that spec (and reject it if it's invalid)

- Then it will store it in etcd

- A *controller* will "notice" that spec and act upon it

---

## Reconciling state

- Watch for the `spec` fields in the YAML files later!

- The *spec* describes *how we want the thing to be*

- Kubernetes will *reconcile* the current state with the spec
  <br/>(technically, this is done by a number of *controllers*)

- When we want to change some resource, we update the *spec*

- Kubernetes will then *converge* that resource

# Exposing containers

- We can connect to our pods using their IP address

- Then we need to figure out a lot of things:

  - how do we look up the IP address of the pod(s)?

  - how do we connect from outside the cluster?

  - how do we load balance traffic?

  - what if a pod fails?

- Kubernetes has a resource type named *Service*

- Services address all these questions!

---

## Running containers with open ports

- Since `ping` doesn't have anything to connect to, we'll have to run something else

- We are going to use `jpetazzo/color`, a tiny HTTP server written in Go

- `jpetazzo/color` listens on port 80

- It serves a page showing the pod's name

  (this will be useful when checking load balancing behavior)

- We could also use the `nginx` official image instead

  (but we wouldn't be able to tell the backends from each other)

---

## Running our HTTP server

- We will create a deployment with `kubectl create deployment`

- This will create a Pod running our HTTP server

- Create a deployment named `blue`:
  ```bash
	pi@pi-red:~ $ kubectl create deployment blue --image=jpetazzo/color
	deployment.apps/blue created
  ```

---

## Connecting to the HTTP server

- Let's connect to the HTTP server directly

  (just to make sure everything works fine; we'll add the Service later)

- Get the IP address of the Pod:
  ```bash
	pi@pi-red:~ $ kubectl get pods -o wide
	NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
	blue-5c986bd7bf-2gf7d       1/1     Running   0          55s   10.244.1.13   pi-black   <none>           <none>
	pingpong-5859ffb968-9fx8c   1/1     Running   0          73m   10.244.1.5    pi-black   <none>           <none>
	pingpong-5859ffb968-fnpw4   1/1     Running   0          64m   10.244.1.12   pi-black   <none>           <none>
	pingpong-5859ffb968-gc2nm   1/1     Running   0          64m   10.244.1.11   pi-black   <none>           <none>
	pingpong-5859ffb968-gpj5p   1/1     Running   0          64m   10.244.1.9    pi-black   <none>           <none>
	pingpong-5859ffb968-j4gbb   1/1     Running   0          64m   10.244.1.8    pi-black   <none>           <none>
	pingpong-5859ffb968-jkr89   1/1     Running   0          71m   10.244.1.7    pi-black   <none>           <none>
	pingpong-5859ffb968-stv9j   1/1     Running   0          64m   10.244.1.10   pi-black   <none>           <none>
	pingpong-5859ffb968-v266p   1/1     Running   0          71m   10.244.1.6    pi-black   <none>           <none>
  ```

- Send an HTTP request to the Pod:
  ```bash
	pi@pi-red:~ $ curl http://10.244.1.13
	🔵This is pod default/blue-5c986bd7bf-2gf7d on linux/arm64, serving / for 10.244.0.0:54312.
  ```

You should see a response from the Pod.

## The Pod doesn't have a "stable identity"

- The IP address that we used above isn't "stable"

  (if the Pod gets deleted, the replacement Pod will have a different address)

- Check the IP addresses of running Pods:
  ```bash
	Every 2.0s: kubectl get pods -o wide                                                                                                                pi-red: Mon Jul 14 23:05:37 2025

	NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
	blue-5c986bd7bf-2gf7d       1/1     Running   0          4m42s   10.244.1.13   pi-black   <none>           <none>
	pingpong-5859ffb968-9fx8c   1/1     Running   0          77m     10.244.1.5    pi-black   <none>           <none>
	pingpong-5859ffb968-fnpw4   1/1     Running   0          68m     10.244.1.12   pi-black   <none>           <none>
	pingpong-5859ffb968-gc2nm   1/1     Running   0          68m     10.244.1.11   pi-black   <none>           <none>
	pingpong-5859ffb968-gpj5p   1/1     Running   0          68m     10.244.1.9    pi-black   <none>           <none>
	pingpong-5859ffb968-j4gbb   1/1     Running   0          68m     10.244.1.8    pi-black   <none>           <none>
	pingpong-5859ffb968-jkr89   1/1     Running   0          75m     10.244.1.7    pi-black   <none>           <none>
	pingpong-5859ffb968-stv9j   1/1     Running   0          68m     10.244.1.10   pi-black   <none>           <none>
	pingpong-5859ffb968-v266p   1/1     Running   0          75m     10.244.1.6    pi-black   <none>           <none>
  ```

- Delete the Pod:
  ```bash
  kubectl delete pod `blue-xxxxxxxx-yyyyy`
	NAME                        READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
	blue-5c986bd7bf-9pnmf       1/1     Running   0          7s    10.244.1.14   pi-black   <none>           <none>
	pingpong-5859ffb968-9fx8c   1/1     Running   0          78m   10.244.1.5    pi-black   <none>           <none>
	pingpong-5859ffb968-fnpw4   1/1     Running   0          69m   10.244.1.12   pi-black   <none>           <none>
	pingpong-5859ffb968-gc2nm   1/1     Running   0          69m   10.244.1.11   pi-black   <none>           <none>
	pingpong-5859ffb968-gpj5p   1/1     Running   0          69m   10.244.1.9    pi-black   <none>           <none>
	pingpong-5859ffb968-j4gbb   1/1     Running   0          69m   10.244.1.8    pi-black   <none>           <none>
	pingpong-5859ffb968-jkr89   1/1     Running   0          76m   10.244.1.7    pi-black   <none>           <none>
	pingpong-5859ffb968-stv9j   1/1     Running   0          69m   10.244.1.10   pi-black   <none>           <none>
	pingpong-5859ffb968-v266p   1/1     Running   0          76m   10.244.1.6    pi-black   <none>           <none>

  ```

- Check that the replacement Pod has a different IP address

## Services in a nutshell

- Services give us a *stable endpoint* to connect to a pod or a group of pods

- An easy way to create a service is to use `kubectl expose`

- If we have a deployment named `my-little-deploy`, we can run:

  `kubectl expose deployment my-little-deploy --port=80`

  ... and this will create a service with the same name (`my-little-deploy`)

- Services are automatically added to an internal DNS zone

  (in the example above, our code can now connect to http://my-little-deploy/)

---

## Exposing our deployment

- Let's create a Service for our Deployment

- Expose the HTTP port of our server:
  ```bash
	pi@pi-red:~ $ kubectl expose deployment blue --port=80
	service/blue exposed
  ```

- Look up which IP address was allocated:
  ```bash
	pi@pi-red:~ $ kubectl get service
	NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
	blue         ClusterIP   10.108.244.161   <none>        80/TCP    16s
	kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   85m
  ```

- By default, this created a `ClusterIP` service

  (we'll discuss later the different types of services)

---

class: extra-details

## Services are layer 4 constructs

- Services can have IP addresses, but they are still *layer 4*

  (i.e. a service is not just an IP address; it's an IP address + protocol + port)

- As a result: you *have to* indicate the port number for your service
    
  (with some exceptions, like `ExternalName` or headless services, covered later)

---

## Testing our service

- We will now send a few HTTP requests to our Pod

- Let's obtain the IP address that was allocated for our service, *programmatically:*
  ```bash
  CLUSTER_IP=$(kubectl get svc blue -o go-template='{{ .spec.clusterIP }}')
  ```

- Send a few requests:
  ```bash
  for i in $(seq 10); do curl http://$CLUSTER_IP; done
  ```

```bash
pi@pi-red:~ $ CLUSTER_IP=$(kubectl get svc blue -o go-template='{{ .spec.clusterIP }}')
pi@pi-red:~ $ echo $CLUSTER_IP
10.108.244.161
pi@pi-red:~ $ for i in $(seq 10); do curl http://$CLUSTER_IP; done
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:15995.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:28485.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:19197.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:13245.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:13203.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:64449.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:42476.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:32250.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:15176.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:39640.
```

---

## A *stable* endpoint

- Let's see what happens when the Pod has a problem

- Keep sending requests to the Service address:
  ```bash
  while sleep 0.3; do curl http://$CLUSTER_IP; done
  ```

- Meanwhile, delete the Pod:
  ```bash
  kubectl delete pod `blue-xxxxxxxx-yyyyy`
  ```

```bash
pi@pi-red:~ $ while sleep 0.3; do curl http://$CLUSTER_IP; done
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:48959.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:38570.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:23597.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:34653.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:36441.
🔵This is pod default/blue-5c986bd7bf-9pnmf on linux/arm64, serving / for 10.244.0.0:63355.
curl: (7) Failed to connect to 10.108.244.161 port 80 after 10219 ms: Couldn't connect to server
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:1270.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:18081.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:35636.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:41162.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:2832.
```

- There might be a short interruption when we delete the pod...

- ...But requests will keep flowing after that (without requiring a manual intervention)

## Load balancing

- The Service will also act as a load balancer

  (if there are multiple Pods in the Deployment)

.lab[

- Scale up the Deployment:
  ```bash
  kubectl scale deployment blue --replicas=3
  ```

- Send a bunch of requests to the Service:
  ```bash
  for i in $(seq 20); do curl http://$CLUSTER_IP; done
  ```

```bash
pi@pi-red:~ $ kubectl scale deployment blue --replicas=3
deployment.apps/blue scaled
pi@pi-red:~ $ for i in $(seq 20); do curl http://$CLUSTER_IP; done
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:54626.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:26874.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:48988.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:52671.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:59716.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:42672.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:3643.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:28934.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:22248.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:28634.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:49399.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:10733.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:7484.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:42592.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:15204.
🔵This is pod default/blue-5c986bd7bf-p7zrl on linux/arm64, serving / for 10.244.0.0:54935.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:5947.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:50553.
🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.0.0:63587.
🔵This is pod default/blue-5c986bd7bf-8ddsb on linux/arm64, serving / for 10.244.0.0:16671.
```

## DNS integration

- Kubernetes provides an internal DNS resolver

- The resolver maps service names to their internal addresses

- By default, this only works *inside Pods* (not from the nodes themselves)

.lab[

- Get a shell in a Pod:
  ```bash
  kubectl run --rm -it --image=fedora test-dns-integration
  ```

- Try to resolve the `blue` Service from the Pod:
  ```bash
	pi@pi-red:~ $ kubectl run --rm -it --image=fedora test-dns-integration
	[root@test-dns-integration /]# curl blue
	🔵This is pod default/blue-5c986bd7bf-987r4 on linux/arm64, serving / for 10.244.1.18:36138.
  ```

## Under the hood...

- Check the content of `/etc/resolv.conf` inside a Pod

- It will have `nameserver X.X.X.X` (e.g. 10.96.0.10)

```bash
[root@test-dns-integration /]# cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

- Now check `kubectl get service kube-dns --namespace=kube-system`

```bash
pi@pi-red:~ $ kubectl get service kube-dns --namespace=kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   98m
```

- ...It's the same address! 😉

- The FQDN of a service is actually:

  `<service-name>.<namespace>.svc.<cluster-domain>`

- `<cluster-domain>` defaults to `cluster.local`

- And the `search` includes `<namespace>.svc.<cluster-domain>`

## Advantages of services

- We don't need to look up the IP address of the pod(s)

  (we resolve the IP address of the service using DNS)

- There are multiple service types; some of them allow external traffic

  (e.g. `LoadBalancer` and `NodePort`)

- Services provide load balancing

  (for both internal and external traffic)

- Service addresses are independent from pods' addresses

  (when a pod fails, the service seamlessly sends traffic to its replacement)

# Service Types

- There are different types of services:

  `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`

- There are also *headless services*

- Services can also have optional *external IPs*

- There is also another resource type called *Ingress*

  (specifically for HTTP services)

- Wow, that's a lot! Let's start with the basics ...

## `ClusterIP`

- It's the default service type

- A virtual IP address is allocated for the service

  (in an internal, private range; e.g. 10.96.0.0/12)

- This IP address is reachable only from within the cluster (nodes and pods)

- Our code can connect to the service using the original port number

- Perfect for internal communication, within the cluster

## `LoadBalancer`

- An external load balancer is allocated for the service

  (typically a cloud load balancer, e.g. ELB on AWS, GLB on GCE ...)

- This is available only when the underlying infrastructure provides some kind of
  "load balancer as a service"

- Each service of that type will typically cost a little bit of money

  (e.g. a few cents per hour on AWS or GCE)

- Ideally, traffic would flow directly from the load balancer to the pods

- In practice, it will often flow through a `NodePort` first

## `NodePort`

- A port number is allocated for the service

  (by default, in the 30000-32767 range)

- That port is made available *on all our nodes* and anybody can connect to it

  (we can connect to any node on that port to reach the service)

- Our code needs to be changed to connect to that new port number

- Under the hood: `kube-proxy` sets up a bunch of `iptables` rules on our nodes

- Sometimes, it's the only available option for external traffic

  (e.g. most clusters deployed with kubeadm or on-premises)

## `ExternalName`

- Services of type `ExternalName` are quite different

- No load balancer (internal or external) is created

- Only a DNS entry gets added to the DNS managed by Kubernetes

- That DNS entry will just be a `CNAME` to a provided record

Example:
```bash
kubectl create service externalname k8s --external-name kubernetes.io
```
*Creates a CNAME `k8s` pointing to `kubernetes.io`*

## External IPs

- We can add an External IP to a service, e.g.:
  ```bash
  kubectl expose deploy my-little-deploy --port=80 --external-ip=1.2.3.4
  ```

- `1.2.3.4` should be the address of one of our nodes

  (it could also be a virtual address, service address, or VIP, shared by multiple nodes)

- Connections to `1.2.3.4:80` will be sent to our service

- External IPs will also show up on services of type `LoadBalancer`

  (they will be added automatically by the process provisioning the load balancer)

## Headless services

- Sometimes, we want to access our scaled services directly:

  - if we want to save a tiny little bit of latency (typically less than 1ms)

  - if we need to connect over arbitrary ports (instead of a few fixed ones)

  - if we need to communicate over another protocol than UDP or TCP

  - if we want to decide how to balance the requests client-side

  - ...

- In that case, we can use a "headless service"


## Creating a headless services

- A headless service is obtained by setting the `clusterIP` field to `None`

  (Either with `--cluster-ip=None`, or by providing a custom YAML)

- As a result, the service doesn't have a virtual IP address

- Since there is no virtual IP address, there is no load balancer either

- CoreDNS will return the pods' IP addresses as multiple `A` records

- This gives us an easy way to discover all the replicas for a deployment

## Services and endpoints

- A service has a number of "endpoints"

- Each endpoint is a host + port where the service is available

- The endpoints are maintained and updated automatically by Kubernetes

- Check the endpoints that Kubernetes has associated with our `blue` service:
  ```bash
	pi@pi-red:~ $ kubectl describe service blue
	Name:                     blue
	Namespace:                default
	Labels:                   app=blue
	Annotations:              <none>
	Selector:                 app=blue
	Type:                     ClusterIP
	IP Family Policy:         SingleStack
	IP Families:              IPv4
	IP:                       10.100.210.2
	IPs:                      10.100.210.2
	Port:                     <unset>  80/TCP
	TargetPort:               80/TCP
	Endpoints:                10.244.1.20:80
	Session Affinity:         None
	Internal Traffic Policy:  Cluster
	Events:                   <none>

	pi@pi-red:~ $ kubectl describe service blue
	Name:                     blue
	Namespace:                default
	Labels:                   app=blue
	Annotations:              <none>
	Selector:                 app=blue
	Type:                     ClusterIP
	IP Family Policy:         SingleStack
	IP Families:              IPv4
	IP:                       10.100.210.2
	IPs:                      10.100.210.2
	Port:                     <unset>  80/TCP
	TargetPort:               80/TCP
	Endpoints:                10.244.1.20:80,10.244.1.21:80,10.244.1.22:80
	Session Affinity:         None
	Internal Traffic Policy:  Cluster
	Events:                   <none>

  ```

In the output, there will be a line starting with `Endpoints:`. for example earlier when we had only one replica for the pod in that case it was only 10.244.1.20:80 and when we scaled the deployment to three replicase then the endpoints are now 10.244.1.20:80,10.244.1.21:80 and 10.244.1.22:80

That line will list a bunch of addresses in `host:port` format.

## Viewing endpoint details

- When we have many endpoints, our display commands truncate the list
  ```bash
	pi@pi-red:~ $ kubectl get endpoints
	Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
	NAME         ENDPOINTS                                      AGE
	blue         10.244.1.20:80,10.244.1.21:80,10.244.1.22:80   5m56s
	kubernetes   192.168.0.152:6443                             13m
  ```

- If we want to see the full list, we can use one of the following commands:
  ```bash
	pi@pi-red:~ $ kubectl describe endpoints blue
	Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
	Name:         blue
	Namespace:    default
	Labels:       app=blue
				endpoints.kubernetes.io/managed-by=endpoint-controller
	Annotations:  endpoints.kubernetes.io/last-change-trigger-time: 2025-07-15T14:57:56Z
	Subsets:
	Addresses:          10.244.1.20,10.244.1.21,10.244.1.22
	NotReadyAddresses:  <none>
	Ports:
		Name     Port  Protocol
		----     ----  --------
		<unset>  80    TCP

	Events:  <none>

	pi@pi-red:~ $ kubectl get endpoints blue -o yaml
	Warning: v1 Endpoints is deprecated in v1.33+; use discovery.k8s.io/v1 EndpointSlice
	apiVersion: v1
	kind: Endpoints
	metadata:
	annotations:
		endpoints.kubernetes.io/last-change-trigger-time: "2025-07-15T14:57:56Z"
	creationTimestamp: "2025-07-15T14:54:54Z"
	labels:
		app: blue
		endpoints.kubernetes.io/managed-by: endpoint-controller
	name: blue
	namespace: default
	resourceVersion: "1495"
	uid: daa03739-09c3-4c6b-af8f-e7e65c171163
	subsets:
	- addresses:
	- ip: 10.244.1.20
		nodeName: pi-black
		targetRef:
		kind: Pod
		name: blue-5c986bd7bf-r7lqd
		namespace: default
		uid: 91907dbe-6529-4d34-8861-1b61adc49a82
	- ip: 10.244.1.21
		nodeName: pi-black
		targetRef:
		kind: Pod
		name: blue-5c986bd7bf-jfsgr
		namespace: default
		uid: 90494f43-caf7-42ee-842c-ed4bc5ab2229
	- ip: 10.244.1.22
		nodeName: pi-black
		targetRef:
		kind: Pod
		name: blue-5c986bd7bf-5gv2d
		namespace: default
		uid: adea4fb5-97b6-4e14-a03f-10de51027815
	ports:
	- port: 80
		protocol: TCP
  ```

- These commands will show us a list of IP addresses

- These IP addresses should match the addresses of the corresponding pods:
  ```bash
	pi@pi-red:~ $ kubectl get pods -l app=blue -o wide
	NAME                    READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
	blue-5c986bd7bf-5gv2d   1/1     Running   0          4m41s   10.244.1.22   pi-black   <none>           <none>
	blue-5c986bd7bf-jfsgr   1/1     Running   0          4m41s   10.244.1.21   pi-black   <none>           <none>
	blue-5c986bd7bf-r7lqd   1/1     Running   0          11m     10.244.1.20   pi-black   <none>           <none>
  ```

## `endpoints` not `endpoint`

- `endpoints` is the only resource that cannot be singular

```bash
pi@pi-red:~ $ kubectl get endpoint
error: the server doesn't have a resource type "endpoint"
```

- This is because the type itself is plural (unlike every other resource)

- There is no `endpoint` object: `type Endpoints struct`

- The type doesn't represent a single endpoint, but a list of endpoints

## `Ingress`

- Ingresses are another type (kind) of resource

- They are specifically for HTTP services

  (not TCP or UDP)

- They can also handle TLS certificates, URL rewriting ...

- They require an *Ingress Controller* to function

## Traffic engineering

- By default, connections to a ClusterIP or a NodePort are load balanced
  across all the backends of their Service

- This can incur extra network hops (which add latency)

- To remove that extra hop, multiple mechanisms are available:

  - `spec.externalTrafficPolicy`

  - `spec.internalTrafficPolicy`

  - [Topology aware routing](https://kubernetes.io/docs/concepts/services-networking/topology-aware-routing/) annotation (beta)

  - `spec.trafficDistribution` (alpha in 1.30, beta in 1.31)

## `internal / externalTrafficPolicy`

- Applies respectively to `ClusterIP` and `NodePort` connections

- Can be set to `Cluster` or `Local`

- `Cluster`: load balance connections across all backends (default)

- `Local`: load balance connections to local backends (on the same node)

- With `Local`, if there is no local backend, the connection will fail!

  (the parameter expresses a "hard rule", not a preference)

- Example: `externalTrafficPolicy: Local` for Ingress controllers

  (as shown on earlier diagrams)

## Topology aware routing

- In beta since Kubernetes 1.23

- Enabled with annotation `service.kubernetes.io/topology-mode=Auto` 

- Relies on node label `topology.kubernetes.io/zone`

- Kubernetes service proxy will try to keep connections within a zone

  (connections made by a pod in zone `a` will be sent to pods in zone `a`)

- ...Except if there are no pods in the zone (then fallback to all zones)

- This can mess up autoscaling!

## `spec.trafficDistribution`

- [KEP4444, Traffic Distribution for Services][kep4444]

- In alpha since Kubernetes 1.30, beta since Kubernetes 1.31

- Should eventually supersede topology aware routing

- Can be set to `PreferClose` (more values might be supported later)

- The meaning of `PreferClose` is implementation dependent

  (with kube-proxy, it should work like topology aware routing: stay in a zone)

[kep4444]: https://github.com/kubernetes/enhancements/issues/4444

# Kubernetes network model

- TL,DR:

  *Our cluster (nodes and pods) is one big flat IP network.*

- In detail:

 - all nodes must be able to reach each other, without NAT

 - all pods must be able to reach each other, without NAT

 - pods and nodes must be able to reach each other, without NAT

 - each pod is aware of its IP address (no NAT)

 - pod IP addresses are assigned by the network implementation

- Kubernetes doesn't mandate any particular implementation

---

## Kubernetes network model: the good

- Everything can reach everything

- No address translation

- No port translation

- No new protocol

- The network implementation can decide how to allocate addresses

- IP addresses don't have to be "portable" from a node to another

  (We can use e.g. a subnet per node and use a simple routed topology)

- The specification is simple enough to allow many various implementations

---

## Kubernetes network model: the less good

- Everything can reach everything

  - if you want security, you need to add network policies

  - the network implementation that you use needs to support them

- There are literally dozens of implementations out there

  (https://github.com/containernetworking/cni/ lists more than 25 plugins)

- Pods have level 3 (IP) connectivity, but *services* are level 4 (TCP or UDP)

  (Services map to a single UDP or TCP port; no port ranges or arbitrary IP packets)

- `kube-proxy` is on the data path when connecting to a pod or container,
  <br/>and it's not particularly fast (relies on userland proxying or iptables)

---

## Kubernetes network model: in practice

- The nodes that we are using have been set up to use [Weave](https://github.com/weaveworks/weave)

- We don't endorse Weave in a particular way, it just Works For Us

- Don't worry about the warning about `kube-proxy` performance

- Unless you:

  - routinely saturate 10G network interfaces
  - count packet rates in millions per second
  - run high-traffic VOIP or gaming platforms
  - do weird things that involve millions of simultaneous connections
    <br/>(in which case you're already familiar with kernel tuning)

- If necessary, there are alternatives to `kube-proxy`; e.g.
  [`kube-router`](https://www.kube-router.io)

## The Container Network Interface (CNI)

- Most Kubernetes clusters use CNI "plugins" to implement networking

- When a pod is created, Kubernetes delegates the network setup to these plugins

  (it can be a single plugin, or a combination of plugins, each doing one task)

- Typically, CNI plugins will:

  - allocate an IP address (by calling an IPAM plugin)

  - add a network interface into the pod's network namespace

  - configure the interface as well as required routes etc.

# Shipping images with a registry

- Initially, our app was running on a single node

- We could *build* and *run* in the same place

- Therefore, we did not need to *ship* anything

- Now that we want to run on a cluster, things are different

- The easiest way to ship container images is to use a registry

## How Docker registries work (a reminder)

- What happens when we execute `docker run alpine` ?

- If the Engine needs to pull the `alpine` image, it expands it into `library/alpine`

- `library/alpine` is expanded into `index.docker.io/library/alpine`

- The Engine communicates with `index.docker.io` to retrieve `library/alpine:latest`

- To use something else than `index.docker.io`, we specify it in the image name

- Examples:
  ```bash
	pi@pi-red:~ $ docker pull gcr.io/google-containers/alpine-with-bash:1.0
	1.0: Pulling from google-containers/alpine-with-bash
	a7fe934dc22a: Pull complete 
	a3ed95caeb02: Pull complete 
	0722d2bf1942: Pull complete 
	Digest: sha256:0955672451201896cf9e2e5ce30bec0c7f10757af33bf78b7a6afa5672c596f5
	Status: Downloaded newer image for gcr.io/google-containers/alpine-with-bash:1.0
	gcr.io/google-containers/alpine-with-bash:1.0
	pi@pi-red:~ $ docker images
	REPOSITORY                                  TAG       IMAGE ID       CREATED        SIZE
	dockercoins-hasher                          latest    bffe12375301   2 weeks ago    330MB
	dockercoins-webui                           latest    dcd8267cb6f5   2 weeks ago    214MB
	dockercoins-worker                          latest    3a66d0a5edb5   2 weeks ago    63.5MB
	dockercoins-rng                             latest    46024a38c668   2 weeks ago    62.2MB
	goravg/instadish                            tag       fa633da41867   3 weeks ago    54.2MB
	<none>                                      <none>    8c161ed7e7d0   5 weeks ago    646MB
	alpine                                      <none>    2abc5e834071   6 weeks ago    8.52MB
	redis                                       latest    c1ae424e8e63   7 weeks ago    150MB
	hello-world                                 latest    f1f77a0f96b7   5 months ago   5.2kB
	gcr.io/google-containers/alpine-with-bash   1.0       822c13824dc2   9 years ago    6.67MB

  docker build -t registry.mycompany.io:5000/myimage:awesome .
  docker push registry.mycompany.io:5000/myimage:awesome
  ```

---

## Running DockerCoins on Kubernetes

- Create one deployment for each component

  (hasher, redis, rng, webui, worker)

- Expose deployments that need to accept connections

  (hasher, redis, rng, webui)

- For redis, we can use the official redis image

- For the 4 others, we need to build images and push them to some registry

---

## Building and shipping images

- There are *many* options!

- Manually:

  - build locally (with `docker build` or otherwise)

  - push to the registry

- Automatically:

  - build and test locally

  - when ready, commit and push a code repository

  - the code repository notifies an automated build system

  - that system gets the code, builds it, pushes the image to the registry

---

## Which registry do we want to use?

- There are SAAS products like Docker Hub, Quay ...

- Each major cloud provider has an option as well

  (ACR on Azure, ECR on AWS, GCR on Google Cloud...)

- There are also commercial products to run our own registry

  (Docker EE, Quay...)

- And open source options, too!

- When picking a registry, pay attention to its build system

  (when it has one)

## Self-hosting our registry

*Note: this section shows how to run the Docker
open source registry and use it to ship images
on our cluster. While this method works fine,
we recommend that you consider using one of the
hosted, free automated build services instead.
It will be much easier!*

*If you need to run a registry on premises,
this section gives you a starting point, but
you will need to make a lot of changes so that
the registry is secured, highly available, and
so that your build pipeline is automated.*

---

## Using the open source registry

- We need to run a `registry` container

- It will store images and layers to the local filesystem
  <br/>(but you can add a config file to use S3, Swift, etc.)

- Docker *requires* TLS when communicating with the registry

  - unless for registries on `127.0.0.0/8` (i.e. `localhost`)

  - or with the Engine flag `--insecure-registry`

- Our strategy: publish the registry container on a NodePort,
  <br/>so that it's available through `127.0.0.1:xxxxx` on each node

---

## Deploying a self-hosted registry

- We will deploy a registry container, and expose it with a NodePort

.lab[

- Create the registry service:
  ```bash
  kubectl create deployment registry --image=registry
  ```

- Expose it on a NodePort:
  ```bash
  kubectl expose deploy/registry --port=5000 --type=NodePort
  ```

## Connecting to our registry

- We need to find out which port has been allocated

- View the service details:
  ```bash
  kubectl describe svc/registry
  ```

- Get the port number programmatically:
  ```bash
  NODEPORT=$(kubectl get svc/registry -o json | jq .spec.ports[0].nodePort)
  REGISTRY=127.0.0.1:$NODEPORT
  ```

---

## Testing our registry

- A convenient Docker registry API route to remember is `/v2/_catalog`

- View the repositories currently held in our registry:
  ```bash
  curl $REGISTRY/v2/_catalog
  ```

--

We should see:
```json
{"repositories":[]}
```

---

## Testing our local registry

- We can retag a small image, and push it to the registry

- Make sure we have the busybox image, and retag it:
  ```bash
  docker pull busybox
  docker tag busybox $REGISTRY/busybox
  ```

- Push it:
  ```bash
  docker push $REGISTRY/busybox
  ```

---

## Checking again what's on our local registry

- Let's use the same endpoint as before

- Ensure that our busybox image is now in the local registry:
  ```bash
  curl $REGISTRY/v2/_catalog
  ```

The curl command should now output:
```json
{"repositories":["busybox"]}
```

```bash
pi@pi-red:~ $ kubectl create deployment registry --image=registry
deployment.apps/registry created
pi@pi-red:~ $ kubectl expose deploy/registry --port=5000 --type=NodePort
service/registry exposed
pi@pi-red:~ $ kubectl describe svc/registry
Name:                     registry
Namespace:                default
Labels:                   app=registry
Annotations:              <none>
Selector:                 app=registry
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.144.151
IPs:                      10.100.144.151
Port:                     <unset>  5000/TCP
TargetPort:               5000/TCP
NodePort:                 <unset>  30470/TCP
Endpoints:                
Session Affinity:         None
External Traffic Policy:  Cluster
Internal Traffic Policy:  Cluster
Events:                   <none>
pi@pi-red:~ $ NODEPORT=$(kubectl get svc/registry -o json | jq .spec.ports[0].nodePort)
REGISTRY=127.0.0.1:$NODEPORT
pi@pi-red:~ $ echo $NODEPORT
30470
pi@pi-red:~ $ curl $REGISTRY/v2/_catalog
{"repositories":[]}
pi@pi-red:~ $ docker pull busybox
Using default tag: latest
latest: Pulling from library/busybox
189fdd150837: Pull complete 
Digest: sha256:f85340bf132ae937d2c2a763b8335c9bab35d6e8293f70f606b9c6178d84f42b
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
pi@pi-red:~ $ docker tag busybox $REGISTRY/busybox
pi@pi-red:~ $ docker push $REGISTRY/busybox
Using default tag: latest
The push refers to repository [127.0.0.1:30470/busybox]
9fa34069244e: Pushed 
latest: digest: sha256:2278daae78b8899e6c7192a8a6f49763b9e0cde6fcf94de34f115df4401b955f size: 527
pi@pi-red:~ $ curl $REGISTRY/v2/_catalog
{"repositories":["busybox"]}
```

## Building and pushing our images

- We are going to use a convenient feature of Docker Compose

- Go to the `stacks` directory:
  ```bash
  cd ~/container.training/stacks
  ```

- Build and push the images:
```bash
pi@pi-red:~/container.training/stacks $ export REGISTRY
pi@pi-red:~/container.training/stacks $ export TAG=0.1
pi@pi-red:~/container.training/stacks $ docker compose -f dockercoins.yml build
WARN[0000] /home/pi/container.training/stacks/dockercoins.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
Compose can now delegate builds to bake for better performance.
 To do so, set COMPOSE_BAKE=true.
[+] Building 146.3s (39/39) FINISHED                                                                                                                                 docker:default
 => [rng internal] load build definition from Dockerfile                                                                                                                       0.1s
 => => transferring dockerfile: 127B                                                                                                                                           0.0s
 => [worker internal] load build definition from Dockerfile                                                                                                                    0.1s
 => => transferring dockerfile: 148B                                                                                                                                           0.0s
 => [hasher internal] load build definition from Dockerfile                                                                                                                    0.1s
 => => transferring dockerfile: 224B                                                                                                                                           0.0s
 => [webui internal] load build definition from Dockerfile                                                                                                                     0.1s
 => => transferring dockerfile: 177B                                                                                                                                           0.0s
 => [rng internal] load metadata for docker.io/library/python:alpine                                                                                                           2.9s
 => [webui internal] load metadata for docker.io/library/node:4-slim                                                                                                           1.7s
 => [hasher internal] load metadata for docker.io/library/ruby:alpine                                                                                                          4.7s
 => [webui internal] load .dockerignore                                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [webui 1/5] FROM docker.io/library/node:4-slim@sha256:26e655b2f02a3a030d52529cc09d4036fe17e270c8b4477bfa17db75ee301405                                                     0.0s
 => [webui internal] load build context                                                                                                                                        0.0s
 => => transferring context: 258B                                                                                                                                              0.0s
 => CACHED [webui 2/5] RUN npm install express@4                                                                                                                               0.0s
 => CACHED [webui 3/5] RUN npm install redis@3                                                                                                                                 0.0s
 => CACHED [webui 4/5] COPY files/ /files/                                                                                                                                     0.0s
 => CACHED [webui 5/5] COPY webui.js /                                                                                                                                         0.0s
 => [webui] exporting to image                                                                                                                                                 0.1s
 => => exporting layers                                                                                                                                                        0.0s
 => => writing image sha256:7f5445c11b55f3e402e6184af63024eccebe7e2dfee8b17b3852e1d7ad8fb31b                                                                                   0.0s
 => => naming to 127.0.0.1:30470/webui:0.1                                                                                                                                     0.0s
 => [webui] resolving provenance for metadata file                                                                                                                             0.0s
 => [rng internal] load .dockerignore                                                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [worker internal] load .dockerignore                                                                                                                                       0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [rng 1/3] FROM docker.io/library/python:alpine@sha256:37b14db89f587f9eaa890e4a442a3fe55db452b69cca1403cc730bd0fbdc8aaf                                                     8.0s
 => => resolve docker.io/library/python:alpine@sha256:37b14db89f587f9eaa890e4a442a3fe55db452b69cca1403cc730bd0fbdc8aaf                                                         0.1s
 => => sha256:2a1e63308567d449f0609869766498f5a8d25ddd56538b8dc7a205590da5d732 5.21kB / 5.21kB                                                                                 0.0s
 => => sha256:6e174226ea690ced550e5641249a412cdbefd2d09871f3e64ab52137a54ba606 4.13MB / 4.13MB                                                                                 3.4s
 => => sha256:d125fdbed4e4ca1d8631d4c952dfbc9fd0aded64865d69eb3c9951e98d6c76fb 449.54kB / 449.54kB                                                                             0.4s
 => => sha256:bcd636e59cfbeb2967d10c7fe5f443b4d16c981fbd523ffee59799ffcaaa8c58 12.55MB / 12.55MB                                                                               5.2s
 => => sha256:37b14db89f587f9eaa890e4a442a3fe55db452b69cca1403cc730bd0fbdc8aaf 10.29kB / 10.29kB                                                                               0.0s
 => => sha256:40c7f36cc642a93b74e1de68f4c1b137db04b5fc2ca8eefe83846015bc129974 1.74kB / 1.74kB                                                                                 0.0s
 => => sha256:2829f301ad519e4a03916d33dfe9077d0c9828693fdac899af12841fa06a3593 248B / 248B                                                                                     0.8s
 => => extracting sha256:6e174226ea690ced550e5641249a412cdbefd2d09871f3e64ab52137a54ba606                                                                                      0.4s
 => => extracting sha256:d125fdbed4e4ca1d8631d4c952dfbc9fd0aded64865d69eb3c9951e98d6c76fb                                                                                      0.3s
 => => extracting sha256:bcd636e59cfbeb2967d10c7fe5f443b4d16c981fbd523ffee59799ffcaaa8c58                                                                                      2.2s
 => => extracting sha256:2829f301ad519e4a03916d33dfe9077d0c9828693fdac899af12841fa06a3593                                                                                      0.0s
 => [rng internal] load build context                                                                                                                                          0.1s
 => => transferring context: 28B                                                                                                                                               0.0s
 => [worker internal] load build context                                                                                                                                       0.1s
 => => transferring context: 31B                                                                                                                                               0.0s
 => [hasher internal] load .dockerignore                                                                                                                                       0.0s
 => => transferring context: 2B                                                                                                                                                0.0s
 => [hasher 1/5] FROM docker.io/library/ruby:alpine@sha256:60d0ffed16d3cfbb3cd42c05f3c3a1c23db85d69716b06895fa54891805a7d65                                                   27.8s
 => => resolve docker.io/library/ruby:alpine@sha256:60d0ffed16d3cfbb3cd42c05f3c3a1c23db85d69716b06895fa54891805a7d65                                                           0.1s
 => => sha256:60d0ffed16d3cfbb3cd42c05f3c3a1c23db85d69716b06895fa54891805a7d65 10.24kB / 10.24kB                                                                               0.0s
 => => sha256:bf8c97bbbb0ce64c7d1c1d0f9386639dcb38ca9779bb32e8ecc9db6dcb94bbb4 1.73kB / 1.73kB                                                                                 0.0s
 => => sha256:4bb51c90b8c7c6181d8f7cb4dcbd922a7a8106d077b976e299845bbf2ec11486 5.84kB / 5.84kB                                                                                 0.0s
 => => sha256:6e174226ea690ced550e5641249a412cdbefd2d09871f3e64ab52137a54ba606 4.13MB / 4.13MB                                                                                 1.6s
 => => sha256:078cb8a2bfe13afc4b6f0d43fa0f60131167c5839b5036be2d4a98c463f0328b 191B / 191B                                                                                     0.3s
 => => sha256:5545c38d7646c487db52f03cd37e75eb6440d7da64ced81db675c915c88bebb3 39.09MB / 39.09MB                                                                              13.6s
 => => extracting sha256:6e174226ea690ced550e5641249a412cdbefd2d09871f3e64ab52137a54ba606                                                                                      0.4s
 => => sha256:3b6797b312992033f598a87cafc9bb4f8aab20e59e342cb2ab508847f3b13874 138B / 138B                                                                                     3.9s
 => => extracting sha256:078cb8a2bfe13afc4b6f0d43fa0f60131167c5839b5036be2d4a98c463f0328b                                                                                      0.0s
 => => extracting sha256:5545c38d7646c487db52f03cd37e75eb6440d7da64ced81db675c915c88bebb3                                                                                      6.0s
 => => extracting sha256:3b6797b312992033f598a87cafc9bb4f8aab20e59e342cb2ab508847f3b13874                                                                                      0.0s
 => [hasher internal] load build context                                                                                                                                       0.0s
 => => transferring context: 31B                                                                                                                                               0.0s
 => [worker 2/4] RUN pip install redis                                                                                                                                        24.2s
 => [rng 2/3] RUN pip install Flask                                                                                                                                           25.8s
 => [hasher 2/5] RUN apk add --update build-base curl                                                                                                                         36.5s
 => [worker 3/4] RUN pip install requests                                                                                                                                     10.9s
 => [rng 3/3] COPY rng.py /                                                                                                                                                    0.8s
 => [rng] exporting to image                                                                                                                                                   1.2s
 => => exporting layers                                                                                                                                                        1.1s
 => => writing image sha256:836ef37963f97125908233ced84993c66d63303949cfe460959054b45ca3b7c1                                                                                   0.0s
 => => naming to 127.0.0.1:30470/rng:0.1                                                                                                                                       0.0s
 => [rng] resolving provenance for metadata file                                                                                                                               0.0s
 => [worker 4/4] COPY worker.py /                                                                                                                                              0.8s
 => [worker] exporting to image                                                                                                                                                0.7s
 => => exporting layers                                                                                                                                                        0.7s
 => => writing image sha256:9e190f44e891d5965a0ea4c473c098521e2efd3b6bc275563963106c425c93c8                                                                                   0.0s
 => => naming to 127.0.0.1:30470/worker:0.1                                                                                                                                    0.0s
 => [worker] resolving provenance for metadata file                                                                                                                            0.0s
 => [hasher 3/5] RUN gem install sinatra --version '~> 3'                                                                                                                      3.7s
 => [hasher 4/5] RUN gem install thin --version '~> 1'                                                                                                                        66.6s
 => [hasher 5/5] ADD hasher.rb /                                                                                                                                               0.2s
 => [hasher] exporting to image                                                                                                                                                6.1s
 => => exporting layers                                                                                                                                                        6.1s
 => => writing image sha256:2d74119566c03c54b41ea5f83701e490580707c296d304118005c67094336e19                                                                                   0.0s
 => => naming to 127.0.0.1:30470/hasher:0.1                                                                                                                                    0.0s
 => [hasher] resolving provenance for metadata file                                                                                                                            0.0s
[+] Building 4/4
 ✔ hasher  Built                                                                                                                                                               0.0s 
 ✔ rng     Built                                                                                                                                                               0.0s 
 ✔ webui   Built                                                                                                                                                               0.0s 
 ✔ worker  Built                                                                                                                                                               0.0s 
pi@pi-red:~/container.training/stacks $ docker compose -f dockercoins.yml push
WARN[0000] /home/pi/container.training/stacks/dockercoins.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Pushing 28/32
 ✔ redis Skipped                                                                                                                                                               0.0s 
 ✔ Pushing 127.0.0.1:30470/worker:0.1: b90dc6830f3e Pushed                                                                                                                     0.6s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: b579a3fad47c Pushed                                                                                                                        0.3s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: a4066965dadf Pushed                                                                                                                        5.7s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: 49a13b4ed03b Pushed                                                                                                                        0.2s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: 6c855cedcf2e Pushed                                                                                                                       14.1s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: b28c4e0aee72 Pushed                                                                                                                        1.0s 
 ✔ Pushing 127.0.0.1:30470/rng:0.1: 0b83d017db6e Pushed                                                                                                                        4.0s 
 ✔ Pushing 127.0.0.1:30470/worker:0.1: 4e9928f3b100 Pushed                                                                                                                     2.5s 
 ✔ Pushing 127.0.0.1:30470/worker:0.1: 4628829de5e2 Pushed                                                                                                                     5.7s 
 ⠋ Pushing 127.0.0.1:30470/worker:0.1: 49a13b4ed03b Mounted from rng                                                                                                          64.0s 
 ✔ Pushing 127.0.0.1:30470/worker:0.1: 6c855cedcf2e Pushed                                                                                                                    15.4s 
 ⠋ Pushing 127.0.0.1:30470/worker:0.1: b28c4e0aee72 Mounted from rng                                                                                                          64.0s 
 ⠋ Pushing 127.0.0.1:30470/worker:0.1: 0b83d017db6e Mounted from rng                                                                                                          64.0s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 429ca6b1b6ad Pushed                                                                                                                      4.9s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 1d49904db8a7 Pushed                                                                                                                      5.7s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: e5e4f7c29386 Pushed                                                                                                                      6.5s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 1d51bbe632cb Pushed                                                                                                                     15.3s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 867f22e49496 Pushed                                                                                                                      7.3s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 2aa7f439e830 Pushed                                                                                                                     25.6s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 3562b882efb0 Pushed                                                                                                                      7.8s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 7abf21970883 Pushed                                                                                                                      8.2s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 10469fb722b4 Pushed                                                                                                                     23.2s 
 ✔ Pushing 127.0.0.1:30470/webui:0.1: 363f5d392123 Pushed                                                                                                                     51.4s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: 894ed3119f43 Pushed                                                                                                                    15.7s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: 29e95bd44111 Pushed                                                                                                                    16.8s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: c10c83b13337 Pushed                                                                                                                    16.8s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: 25bcd38b2eb4 Pushed                                                                                                                    63.8s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: d2a16db07749 Pushed                                                                                                                    17.1s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: 2e4f320cd395 Pushed                                                                                                                    40.7s 
 ✔ Pushing 127.0.0.1:30470/hasher:0.1: e4dbfc7260a3 Pushed                                                                                                                    23.5s 
 ⠋ Pushing 127.0.0.1:30470/hasher:0.1: 0b83d017db6e Mounted from worker                                                                                                       64.0s 

```

Let's have a look at the `dockercoins.yml` file while this is building and pushing.

```yaml
version: "3"

services:
  rng:
    build: dockercoins/rng
    image: ${REGISTRY-127.0.0.1:5000}/rng:${TAG-latest}
    deploy:
      mode: global
  ...
  redis:
    image: redis
  ...
  worker:
    build: dockercoins/worker
    image: ${REGISTRY-127.0.0.1:5000}/worker:${TAG-latest}
    ...
    deploy:
      replicas: 10
```

## Avoiding the `latest` tag

.warning[Make sure that you've set the `TAG` variable properly!]

- If you don't, the tag will default to `latest`

- The problem with `latest`: nobody knows what it points to!

  - the latest commit in the repo?

  - the latest commit in some branch? (Which one?)

  - the latest tag?

  - some random version pushed by a random team member?

- If you keep pushing the `latest` tag, how do you roll back?

- Image tags should be meaningful, i.e. correspond to code branches, tags, or hashes

## Checking the content of the registry

- All our images should now be in the registry

- Re-run the same `curl` command as earlier:
  ```bash
	pi@pi-red:~/container.training/stacks $ curl $REGISTRY/v2/_catalog
	{"repositories":["busybox","hasher","rng","webui","worker"]}
  ```

```
In these slides, all the commands to deploy
DockerCoins will use a $REGISTRY environment
variable, so that we can quickly switch from
the self-hosted registry to pre-built images
hosted on the Docker Hub. So make sure that
this $REGISTRY variable is set correctly when
running these commands!
```

# Running our application on Kubernetes

- We can now deploy our code (as well as a redis instance)

- Deploy `redis`:
  ```bash
	pi@pi-red:~/container.training/stacks $ kubectl create deployment redis --image=redis
	deployment.apps/redis created
  ```

- Deploy everything else:
  ```bash
	pi@pi-red:~/container.training/stacks $ kubectl create deployment hasher --image=dockercoins/hasher:v0.1
	deployment.apps/hasher created
	pi@pi-red:~/container.training/stacks $ kubectl create deployment rng --image=dockercoins/rng:v0.1
	deployment.apps/rng created
	pi@pi-red:~/container.training/stacks $ kubectl create deployment webui --image=dockercoins/webui:v0.1
	deployment.apps/webui created
	pi@pi-red:~/container.training/stacks $ kubectl create deployment worker --image=dockercoins/worker:v0.1
	deployment.apps/worker created
  ```

---

class: extra-details

## Deploying other images

- If we wanted to deploy images from another registry ...

- ... Or with a different tag ...

- ... We could use the following snippet:

```bash
  REGISTRY=dockercoins
  TAG=v0.1
  for SERVICE in hasher rng webui worker; do
    kubectl create deployment $SERVICE --image=$REGISTRY/$SERVICE:$TAG
  done
```

---

## Is this working?

- After waiting for the deployment to complete, let's look at the logs!

  (Hint: use `kubectl get deploy -w` to watch deployment events)


- Look at some logs:
  ```bash
	pi@pi-red:~ $ kubectl logs deploy/rng
	* Serving Flask app 'rng' (lazy loading)
	* Environment: production
	WARNING: This is a development server. Do not use it in a production deployment.
	Use a production WSGI server instead.
	* Debug mode: off
	* Running on all addresses.
	WARNING: This is a development server. Do not use it in a production deployment.
	* Running on http://10.244.1.27:80/ (Press CTRL+C to quit)

	pi@pi-red:~ $ kubectl logs deploy/worker
	Error from server (BadRequest): container "worker" in pod "worker-5c6f84b477-nrh7g" is waiting to start: ContainerCreating
  ```

🤔 `rng` is fine ... But not `worker`.

💡 Oh right! We forgot to `expose`.

---

## Connecting containers together

- Three deployments need to be reachable by others: `hasher`, `redis`, `rng`

- `worker` doesn't need to be exposed

- `webui` will be dealt with later

- Expose each deployment, specifying the right port:
  ```bash
	pi@pi-red:~ $ kubectl expose deployment redis --port 6379
	service/redis exposed
	pi@pi-red:~ $ kubectl expose deployment rng --port 80
	service/rng exposed
	pi@pi-red:~ $ kubectl expose deployment hasher --port 80
	service/hasher exposed
  ```

---

## Is this working yet?

- The `worker` has an infinite loop, that retries 10 seconds after an error

- Stream the worker's logs:
  ```bash
  kubectl logs deploy/worker --follow
  ```

  (Give it about 10 seconds to recover)

<!--
```wait units of work done, updating hash counter```
```key ^C```
-->

]

--

We should now see the `worker`, well, working happily.

---

## Exposing services for external access

- Now we would like to access the Web UI

- We will expose it with a `NodePort`

  (just like we did for the registry)

.lab[

- Create a `NodePort` service for the Web UI:
  ```bash
	pi@pi-red:~ $ kubectl expose deploy/webui --type=NodePort --port=80
	service/webui exposed
  ```

- Check the port that was allocated:

  ```bash
	pi@pi-red:~ $ kubectl get svc
	NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
	hasher       ClusterIP   10.108.177.207   <none>        80/TCP           2m8s
	kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          41m
	redis        ClusterIP   10.106.12.142    <none>        6379/TCP         2m17s
	registry     NodePort    10.100.144.151   <none>        5000:30470/TCP   27m
	rng          ClusterIP   10.111.60.121    <none>        80/TCP           2m13s
	webui        NodePort    10.109.152.138   <none>        80:30088/TCP     29s
  ```

## Accessing the web UI

- We can now connect to *any node*, on the allocated node port, to view the web UI

```bash
pi@pi-red:~ $ kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                         KERNEL-VERSION       CONTAINER-RUNTIME
pi-black   Ready    <none>          42m   v1.33.2   192.168.0.101   <none>        Debian GNU/Linux 12 (bookworm)   6.12.25+rpt-rpi-v8   containerd://1.7.27
pi-red     Ready    control-plane   43m   v1.33.2   192.168.0.152   <none>        Debian GNU/Linux 12 (bookworm)   6.6.51+rpt-rpi-v8    containerd://1.7.27
```

- Open the web UI in your browser (http://192.168.0.152:30088)

Yes, this may take a little while to update. *(Narrator: it was DNS.)*


*Alright, we're back to where we started, when we were running on a single node!*

# Gentle introduction to YAML

- YAML Ain't Markup Language (according to [yaml.org][yaml])

- *Almost* required when working with containers:

  - Docker Compose files

  - Kubernetes manifests

  - Many CI pipelines (GitHub, GitLab...)

- If you don't know much about YAML, this is for you!

[yaml]: https://yaml.org/

---

## What is it?

- Data representation language

```yaml
- country: France
  capital: Paris
  code: fr
  population: 68042591
- country: Germany
  capital: Berlin
  code: de
  population: 84270625
- country: Norway
  capital: Oslo
  code: no # It's a trap!
  population: 5425270
```

- Even without knowing YAML, we probably can add a country to that file :)

---

## Trying YAML

- Method 1: in the browser

  https://onlineyamltools.com/convert-yaml-to-json

  https://onlineyamltools.com/highlight-yaml

- Method 2: in a shell

  ```bash
  yq . foo.yaml
  ```

- Method 3: in Python

  ```python
    import yaml; yaml.safe_load("""
    - country: France
      capital: Paris
    """)
  ```

---

## Basic stuff

- Strings, numbers, boolean values, `null`

- Sequences (=arrays, lists)

- Mappings (=objects)

- Superset of JSON

  (if you know JSON, you can just write JSON)

- Comments start with `#`

- A single *file* can have multiple *documents*

  (separated by `---` on a single line)

---

## Sequences

- Example: sequence of strings
  ```yaml
  [ "france", "germany", "norway" ]
  ```

- Example: the same sequence, without the double-quotes
  ```yaml
  [ france, germany, norway ]
  ```

- Example: the same sequence, in "block collection style" (=multi-line)
  ```yaml
  - france
  - germany
  - norway
  ```

---

## Mappings

- Example: mapping strings to numbers
  ```yaml
  { "france": 68042591, "germany": 84270625, "norway": 5425270 }
  ```

- Example: the same mapping, without the double-quotes
  ```yaml
  { france: 68042591, germany: 84270625, norway: 5425270 }
  ```

- Example: the same mapping, in "block collection style"
  ```yaml
  france: 68042591
  germany: 84270625
  norway: 5425270
  ```

---

## Combining types

- In a sequence (or mapping) we can have different types

  (including other sequences or mappings)

- Example:
  ```yaml
  questions: [ name, quest, favorite color ]
  answers: [ "Arthur, King of the Britons", Holy Grail, purple, 42 ]
  ```

- Note that we need to quote "Arthur" because of the comma

- Note that we don't have the same number of elements in questions and answers

---

## More combinations

- Example:
  ```yaml
    - service: nginx
      ports: [ 80, 443 ]
    - service: bind
      ports: [ 53/tcp, 53/udp ]
    - service: ssh
      ports: 22
  ```

- Note that `ports` doesn't always have the same type

  (the code handling that data will probably have to be smart!)

---

## ⚠️ Automatic booleans

```yaml
codes:
  france: fr
  germany: de
  norway: no
```

--

```json
{
  "codes": {
    "france": "fr",
    "germany": "de",
    "norway": false
  }
}
```

---

## ⚠️ Automatic booleans

- `no` can become `false`

  (it depends on the YAML parser used)

- It should be quoted instead:

  ```yaml
    codes:
      france: fr
      germany: de
      norway: "no"
  ```

---

## ⚠️ Automatic floats

```yaml
version:
  libfoo: 1.10
  fooctl: 1.0
```

--

```json
{
  "version": {
    "libfoo": 1.1,
    "fooctl": 1
  }
}
```

---

## ⚠️ Automatic floats

- Trailing zeros disappear

- These should also be quoted:

  ```yaml
    version:
      libfoo: "1.10"
      fooctl: "1.0"
  ```

---

## ⚠️ Automatic times

```yaml
portmap:
- 80:80
- 22:22
```

--

```json
{
  "portmap": [
    "80:80",
    1342
  ]
}
```

---

## ⚠️ Automatic times

- `22:22` becomes `1342`

- Thats 22 minutes and 22 seconds = 1342 seconds

- Again, it should be quoted

---

## Document separator

- A single YAML *file* can have multiple *documents* separated by `---`:

  ```yaml
    This is a document consisting of a single string.
    --- 💡
    name: The second document
    type: This one is a mapping (key→value)
    --- 💡
    - Third document
    - This one is a sequence
  ```

- Some folks like to add an extra `---` at the beginning and/or at the end

  (it's not mandatory but can help e.g. to `cat` multiple files together)

.footnote[💡 Ignore this; it's here to work around [this issue][remarkyaml].]

[remarkyaml]: https://github.com/gnab/remark/issues/679

---

## Multi-line strings

Try the following block in a YAML parser:

```yaml
add line breaks: "in double quoted strings\n(like this)"
preserve line break: |
  by using
  a pipe (|)
  (this is great for embedding shell scripts, configuration files...)
do not preserve line breaks: >
  by using
  a greater-than (>)
  (this is great for embedding very long lines)
```

See https://yaml-multiline.info/ for advanced multi-line tips!

(E.g. to strip or keep extra `\n` characters at the end of the block.)

---

class: extra-details

## Advanced features

Anchors let you "memorize" and re-use content:

```yaml
debian: &debian
  packages: deb
  latest-stable: bullseye

also-debian: *debian

ubuntu:
  <<: *debian
  latest-stable: jammy
```

---

class: extra-details

## YAML, good or evil?

- Natural progression from XML to JSON to YAML

- There are other data languages out there

  (e.g. HCL, domain-specific things crafted with Ruby, CUE...)

- Compromises are made, for instance:

  - more user-friendly → more "magic" with side effects

  - more powerful → steeper learning curve

- Love it or loathe it but it's a good idea to understand it!

- Interesting tool if you appreciate YAML: https://carvel.dev/ytt/

# Deploying with YAML

- So far, we created resources with the following commands:

  - `kubectl run`

  - `kubectl create deployment`

  - `kubectl expose`

- We can also create resources directly with YAML manifests

## Why use YAML? (1/3)

- Some resources cannot be created easily with `kubectl`

  (e.g. DaemonSets, StatefulSets, webhook configurations...)

- Some features and fields aren't directly available

  (e.g. resource limits, healthchecks, volumes...)

## Why use YAML? (2/3)

- Create a complicated resource with a single, simple command:

  `kubectl create -f stuff.yaml`

- Create *multiple* resources with a single, simple command:

  `kubectl create -f more-stuff.yaml` or `kubectl create -f directory-with-yaml/`

- Create resources from a remote manifest:

  `kubectl create -f https://.../.../stuff.yaml`

- Create and update resources:

  `kubectl apply -f stuff.yaml`

## Why use YAML? (3/3)

- YAML lets us work *declaratively*

- Describe what we want to deploy/run on Kubernetes

  ("desired state")

- Use tools like `kubectl`, Helm, kapp, Flux, ArgoCD... to make it happen

  ("reconcile" actual state with desired state)

- Very similar to e.g. Terraform

## Overrides and `kubectl set`

Just so you know...

- `kubectl create deployment ... --overrides '{...}'`

  *specify a patch that will be applied on top of the YAML generated by `kubectl`*

- `kubectl set ...`

  *lets us change e.g. images, service accounts, resources, and much more*

## Various ways to write YAML

- From examples in the docs, tutorials, blog posts, LLMs...

  (easiest option when getting started)

- Dump an existing resource with `kubectl get -o yaml ...`

  (includes many extra fields; it is recommended to clean up the result)

- Ask `kubectl` to generate the YAML

  (with `kubectl --dry-run=client -o yaml create/run ...`)

- Completely from scratch with our favorite editor

  (black belt level😅)

## Writing a Pod manifest

- Let's use `kubectl --dry-run=client -o yaml`

- Generate the Pod manifest:
  ```bash
	pi@pi-red:~ $ kubectl run --dry-run=client -o yaml purple --image=jpetazzo/color
	apiVersion: v1
	kind: Pod
	metadata:
	creationTimestamp: null
	labels:
		run: purple
	name: purple
	spec:
	containers:
	- image: jpetazzo/color
		name: purple
		resources: {}
	dnsPolicy: ClusterFirst
	restartPolicy: Always
	status: {}
  ```

- Save it to a file:
  ```bash
	pi@pi-red:~ $ kubectl run --dry-run=client -o yaml purple --image=jpetazzo/color \
	> pod-purple.yaml
	pi@pi-red:~ $ ls
	container.training  k8-init.sh  k8s-install.sh  master.sh  myenv  out.dat  out.png  pingpong.yaml  pod-purple.yaml
	pi@pi-red:~ $ cat pod-purple.yaml
	apiVersion: v1
	kind: Pod
	metadata:
	creationTimestamp: null
	labels:
		run: purple
	name: purple
	spec:
	containers:
	- image: jpetazzo/color
		name: purple
		resources: {}
	dnsPolicy: ClusterFirst
	restartPolicy: Always
	status: {}
  ```

## Running the Pod

- Let's create the Pod with the manifest we just generated

.lab[

- Create all the resources (at this point, just our Pod) described in the manifest:
  ```bash
	pi@pi-red:~ $ kubectl create -f pod-purple.yaml
	pod/purple created
  ```

- Confirm that the Pod is running
  ```bash
	pi@pi-red:~ $ kubectl get pods
	NAME                        READY   STATUS    RESTARTS   AGE
	hasher-99bbd4bb-h955v       1/1     Running   0          63m
	pingpong-5859ffb968-h5qmm   1/1     Running   0          96m
	purple                      1/1     Running   0          25s
	redis-7b47f84cc4-975m7      1/1     Running   0          63m
	registry-848598b5b6-pbkgj   1/1     Running   0          86m
	rng-65d885d498-lcw97        1/1     Running   0          63m
	webui-74bb6bbc59-d4sft      1/1     Running   0          63m
	worker-5c6f84b477-nrh7g     1/1     Running   0          63m
  ```

## Comparing with direct `kubectl run`

- The Pod should be identical to one created directly with `kubectl run`

.lab[

- Create a Pod directly with `kubectl run`:
  ```bash
	pi@pi-red:~ $ kubectl run yellow --image=jpetazzo/color
	pod/yellow created
  ```

- Compare both Pod manifests and status:
  ```bash
	pi@pi-red:~ $ kubectl get pod purple -o yaml
	apiVersion: v1
	kind: Pod
	metadata:
	creationTimestamp: "2025-07-18T16:00:42Z"
	generation: 1
	labels:
		run: purple
	name: purple
	namespace: default
	resourceVersion: "9101"
	uid: a34c6b61-255f-4137-80e1-9b024e2dfc6d
	spec:
	containers:
	- image: jpetazzo/color
		imagePullPolicy: Always
		name: purple
		resources: {}
		terminationMessagePath: /dev/termination-log
		terminationMessagePolicy: File
		volumeMounts:
		- mountPath: /var/run/secrets/kubernetes.io/serviceaccount
		name: kube-api-access-4jzjz
		readOnly: true
	dnsPolicy: ClusterFirst
	enableServiceLinks: true
	nodeName: pi-black
	preemptionPolicy: PreemptLowerPriority
	priority: 0
	restartPolicy: Always
	schedulerName: default-scheduler
	securityContext: {}
	serviceAccount: default
	serviceAccountName: default
	terminationGracePeriodSeconds: 30
	tolerations:
	- effect: NoExecute
		key: node.kubernetes.io/not-ready
		operator: Exists
		tolerationSeconds: 300
	- effect: NoExecute
		key: node.kubernetes.io/unreachable
		operator: Exists
		tolerationSeconds: 300
	volumes:
	- name: kube-api-access-4jzjz
		projected:
		defaultMode: 420
		sources:
		- serviceAccountToken:
			expirationSeconds: 3607
			path: token
		- configMap:
			items:
			- key: ca.crt
				path: ca.crt
			name: kube-root-ca.crt
		- downwardAPI:
			items:
			- fieldRef:
				apiVersion: v1
				fieldPath: metadata.namespace
				path: namespace
	status:
	conditions:
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:00:46Z"
		status: "True"
		type: PodReadyToStartContainers
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:00:42Z"
		status: "True"
		type: Initialized
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:00:46Z"
		status: "True"
		type: Ready
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:00:46Z"
		status: "True"
		type: ContainersReady
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:00:42Z"
		status: "True"
		type: PodScheduled
	containerStatuses:
	- containerID: containerd://5b044d0afda1060f38c6c07b02c4eed9386278986dac56d87e86494546e6d966
		image: docker.io/jpetazzo/color:latest
		imageID: docker.io/jpetazzo/color@sha256:f4c74129c53db381b8eedd37ec0e6669ebff970308ce173630e80c5c5ecff755
		lastState: {}
		name: purple
		ready: true
		resources: {}
		restartCount: 0
		started: true
		state:
		running:
			startedAt: "2025-07-18T16:00:45Z"
		volumeMounts:
		- mountPath: /var/run/secrets/kubernetes.io/serviceaccount
		name: kube-api-access-4jzjz
		readOnly: true
		recursiveReadOnly: Disabled
	hostIP: 192.168.0.101
	hostIPs:
	- ip: 192.168.0.101
	phase: Running
	podIP: 10.244.1.30
	podIPs:
	- ip: 10.244.1.30
	qosClass: BestEffort
	startTime: "2025-07-18T16:00:42Z"
	pi@pi-red:~ $ kubectl get pod yellow -o yaml
	apiVersion: v1
	kind: Pod
	metadata:
	creationTimestamp: "2025-07-18T16:01:47Z"
	generation: 1
	labels:
		run: yellow
	name: yellow
	namespace: default
	resourceVersion: "9200"
	uid: 317f87f7-04f0-4d3b-ab79-fa060f9979bd
	spec:
	containers:
	- image: jpetazzo/color
		imagePullPolicy: Always
		name: yellow
		resources: {}
		terminationMessagePath: /dev/termination-log
		terminationMessagePolicy: File
		volumeMounts:
		- mountPath: /var/run/secrets/kubernetes.io/serviceaccount
		name: kube-api-access-5kvql
		readOnly: true
	dnsPolicy: ClusterFirst
	enableServiceLinks: true
	nodeName: pi-black
	preemptionPolicy: PreemptLowerPriority
	priority: 0
	restartPolicy: Always
	schedulerName: default-scheduler
	securityContext: {}
	serviceAccount: default
	serviceAccountName: default
	terminationGracePeriodSeconds: 30
	tolerations:
	- effect: NoExecute
		key: node.kubernetes.io/not-ready
		operator: Exists
		tolerationSeconds: 300
	- effect: NoExecute
		key: node.kubernetes.io/unreachable
		operator: Exists
		tolerationSeconds: 300
	volumes:
	- name: kube-api-access-5kvql
		projected:
		defaultMode: 420
		sources:
		- serviceAccountToken:
			expirationSeconds: 3607
			path: token
		- configMap:
			items:
			- key: ca.crt
				path: ca.crt
			name: kube-root-ca.crt
		- downwardAPI:
			items:
			- fieldRef:
				apiVersion: v1
				fieldPath: metadata.namespace
				path: namespace
	status:
	conditions:
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:01:50Z"
		status: "True"
		type: PodReadyToStartContainers
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:01:47Z"
		status: "True"
		type: Initialized
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:01:50Z"
		status: "True"
		type: Ready
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:01:50Z"
		status: "True"
		type: ContainersReady
	- lastProbeTime: null
		lastTransitionTime: "2025-07-18T16:01:47Z"
		status: "True"
		type: PodScheduled
	containerStatuses:
	- containerID: containerd://25ac7ea9b7e7b98ed3bd5f9e6f08c68b6e453ae37ee905513717bdd87052eedf
		image: docker.io/jpetazzo/color:latest
		imageID: docker.io/jpetazzo/color@sha256:f4c74129c53db381b8eedd37ec0e6669ebff970308ce173630e80c5c5ecff755
		lastState: {}
		name: yellow
		ready: true
		resources: {}
		restartCount: 0
		started: true
		state:
		running:
			startedAt: "2025-07-18T16:01:50Z"
		volumeMounts:
		- mountPath: /var/run/secrets/kubernetes.io/serviceaccount
		name: kube-api-access-5kvql
		readOnly: true
		recursiveReadOnly: Disabled
	hostIP: 192.168.0.101
	hostIPs:
	- ip: 192.168.0.101
	phase: Running
	podIP: 10.244.1.31
	podIPs:
	- ip: 10.244.1.31
	qosClass: BestEffort
	startTime: "2025-07-18T16:01:47Z"
  ```

## Generating a Deployment manifest

- After a Pod, let's create a Deployment!

- Generate the YAML for a Deployment:
  ```bash
	pi@pi-red:~ $ kubectl create deployment purple --image=jpetazzo/color -o yaml --dry-run=client
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	replicas: 1
	selector:
		matchLabels:
		app: purple
	strategy: {}
	template:
		metadata:
		creationTimestamp: null
		labels:
			app: purple
		spec:
		containers:
		- image: jpetazzo/color
			name: color
			resources: {}
	status: {}
  ```

- Save it to a file:
  ```bash
	pi@pi-red:~ $ kubectl create deployment purple --image=jpetazzo/color -o yaml --dry-run=client \
	> deployment-purple.yaml
	pi@pi-red:~ $ cat deployment-purple.yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	replicas: 1
	selector:
		matchLabels:
		app: purple
	strategy: {}
	template:
		metadata:
		creationTimestamp: null
		labels:
			app: purple
		spec:
		containers:
		- image: jpetazzo/color
			name: color
			resources: {}
	status: {}
  ```

- And create the Deployment:
  ```bash
	pi@pi-red:~ $ kubectl create -f deployment-purple.yaml
	deployment.apps/purple created
  ```

## Updating our Deployment

- What if we want to scale that Deployment?

- Option 1: `kubectl scale`

- Option 2: update the YAML manifest

- Let's go with option 2!

- Edit the YAML manifest:
  ```bash
  vim deployment-purple.yaml
  ```

- Find the line with `replicas: 1` and update the number of replicas

```bash
pi@pi-red:~ $ vim deployment-purple.yaml
pi@pi-red:~ $ cat deployment-purple.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: purple
  name: purple
spec:
  replicas: 10
  selector:
    matchLabels:
      app: purple
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: purple
    spec:
      containers:
      - image: jpetazzo/color
        name: color
        resources: {}
status: {}
```

## Applying our changes

- Problem: `kubectl create` won't update ("overwrite") resources

- Try it out:
  ```bash
	pi@pi-red:~ $ kubectl create -f deployment-purple.yaml
	Error from server (AlreadyExists): error when creating "deployment-purple.yaml": deployments.apps "purple" already exists
  ```

- So, what can we do?

## Updating resources

- Option 1: delete the Deployment and re-create it

  (effective, but causes downtime!)

- Option 2: `kubectl scale` or `kubectl edit` the Deployment

  (effective, but that's cheating - we want to use YAML!)

- Option 3: `kubectl apply`

---

## `kubectl apply` vs `create`

- `kubectl create -f whatever.yaml`

  - creates resources if they don't exist

  - if resources already exist, don't alter them
    <br/>(and display error message)

- `kubectl apply -f whatever.yaml`

  - creates resources if they don't exist

  - if resources already exist, update them
    <br/>(to match the definition provided by the YAML file)

  - stores the manifest as an *annotation* in the resource

---

## Trying `kubectl apply`

.lab[

- First, delete the Deployment:
  ```bash
  kubectl delete deployment purple
  ```

- Re-create it using `kubectl apply`:
  ```bash
  kubectl apply -f deployment-purple.yaml
  ```

- Edit the YAML manifest, change the number of replicas again:
  ```bash
  vim deployment-purple.yaml
  ```

- Apply the new manifest:
  ```bash
  kubectl apply -f deployment-purple.yaml
  ```

## `create` → `apply`

- What are the differences between `kubectl create -f` an `kubectl apply -f`?

  - `kubectl apply` adds an annotation
    <br/>
    (`kubectl.kubernetes.io/last-applied-configuration`)

  - `kubectl apply` makes an extra `GET` request
    <br/>
    (to get the existing object, or at least check if there is one)

- Otherwise, the end result is the same!

- It's almost always better to use `kubectl apply`

  (except when we don't want the extra annotation, e.g. for huge objects like some CRDs)

- From now on, we'll almost always use `kubectl apply -f` instead of `kubectl create -f`

```bash
pi@pi-red:~ $ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
hasher-99bbd4bb-h955v       1/1     Running   0          75m
pingpong-5859ffb968-h5qmm   1/1     Running   0          108m
purple                      1/1     Running   0          12m
purple-65bb9bc655-lg4bd     1/1     Running   0          7m33s
redis-7b47f84cc4-975m7      1/1     Running   0          75m
registry-848598b5b6-pbkgj   1/1     Running   0          98m
rng-65d885d498-lcw97        1/1     Running   0          75m
webui-74bb6bbc59-d4sft      1/1     Running   0          75m
worker-5c6f84b477-nrh7g     1/1     Running   0          75m
yellow                      1/1     Running   0          11m
pi@pi-red:~ $ kubectl apply -f deployment-purple.yaml
Warning: resource deployments/purple is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/purple configured
pi@pi-red:~ $ kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
hasher-99bbd4bb-h955v       1/1     Running             0          75m
pingpong-5859ffb968-h5qmm   1/1     Running             0          108m
purple                      1/1     Running             0          12m
purple-65bb9bc655-2n52k     0/1     ContainerCreating   0          3s
purple-65bb9bc655-9t6pw     0/1     ContainerCreating   0          3s
purple-65bb9bc655-bq22k     0/1     ContainerCreating   0          3s
purple-65bb9bc655-dwfpf     0/1     ContainerCreating   0          3s
purple-65bb9bc655-lbbjb     0/1     ContainerCreating   0          3s
purple-65bb9bc655-lg4bd     1/1     Running             0          7m44s
purple-65bb9bc655-p6spr     0/1     ContainerCreating   0          3s
purple-65bb9bc655-pvxd4     0/1     ContainerCreating   0          3s
purple-65bb9bc655-whkx4     0/1     ContainerCreating   0          3s
purple-65bb9bc655-zk95j     0/1     ContainerCreating   0          3s
redis-7b47f84cc4-975m7      1/1     Running             0          76m
registry-848598b5b6-pbkgj   1/1     Running             0          98m
rng-65d885d498-lcw97        1/1     Running             0          75m
webui-74bb6bbc59-d4sft      1/1     Running             0          75m
worker-5c6f84b477-nrh7g     1/1     Running             0          75m
yellow                      1/1     Running             0          11m
```

- Here initially there was only 1 replica for purple was configured in yaml configuration file but now we have changed file using vim and then appied the configuration using kubectl apply -f `<filename>`

## Adding a Service

- Let's generate the YAML for a Service exposing our Deployment

.lab[

- Run `kubectl expose`, once again with `-o yaml --dry-run=client`:
  ```bash
	pi@pi-red:~ $ kubectl expose deployment purple --port 80 -o yaml --dry-run=client
	apiVersion: v1
	kind: Service
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	ports:
	- port: 80
		protocol: TCP
		targetPort: 80
	selector:
		app: purple
	status:
	loadBalancer: {}
  ```

- Save it to a file:
  ```bash
	pi@pi-red:~ $ kubectl expose deployment purple --port 80 -o yaml --dry-run=client \
	> service-purple.yaml
	pi@pi-red:~ $ cat service-purple.yaml
	apiVersion: v1
	kind: Service
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	ports:
	- port: 80
		protocol: TCP
		targetPort: 80
	selector:
		app: purple
	status:
	loadBalancer: {}
  ```

- Note: if the Deployment doesn't exist, `kubectl expose` won't work!

## What if the Deployment doesn't exist?

- We can also use `kubectl create service`

- The syntax is slightly different

  (`--port` becomes `--tcp` for some reason)

.lab[

- Generate the YAML with `kubectl create service`:
  ```bash
	pi@pi-red:~ $ kubectl create service clusterip purple --tcp 80 -o yaml --dry-run=client
	apiVersion: v1
	kind: Service
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	ports:
	- name: "80"
		port: 80
		protocol: TCP
		targetPort: 80
	selector:
		app: purple
	type: ClusterIP
	status:
	loadBalancer: {}
  ```

## Combining manifests

- We can put multiple resources in a single YAML file

- We need to separate them with the standard YAML document separator

  (i.e. `---` standing by itself on a single line)

.lab[

- Generate a combined YAML file:
  ```bash
	pi@pi-red:~ $ for YAMLFILE in deployment-purple.yaml service-purple.yaml; do
		echo ---
		cat $YAMLFILE
		done > app-purple.yaml
	pi@pi-red:~ $ cat app-purple.yaml
	---
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	replicas: 10
	selector:
		matchLabels:
		app: purple
	strategy: {}
	template:
		metadata:
		creationTimestamp: null
		labels:
			app: purple
		spec:
		containers:
		- image: jpetazzo/color
			name: color
			resources: {}
	status: {}
	---
	apiVersion: v1
	kind: Service
	metadata:
	creationTimestamp: null
	labels:
		app: purple
	name: purple
	spec:
	ports:
	- port: 80
		protocol: TCP
		targetPort: 80
	selector:
		app: purple
	status:
	loadBalancer: {}
  ```

## Resource ordering

- *In general,* the order of the resources doesn't matter:

  - in many cases, resources don't reference each other explicitly
    <br/>
    (e.g. a Service can exist even if the corresponding Deployment doesn't)

  - in some cases, there might be a transient error, but Kubernetes will retry
    <br/>
    (and eventually succeed)

- One exception: Namespaces should be created *before* resources in them!

## Using `-f` with other commands

- We can also use `kubectl delete -f`, `kubectl label -f`, and more!

.lab[

- Apply the resulting YAML file:
  ```bash
  kubectl apply -f app-purple.yaml
  ```

- Add a label to both the Deployment and the Service:
  ```bash
  kubectl label -f app-purple.yaml release=production
  ```

- Delete them:
  ```bash
  kubectl delete -f app-purple.yaml
  ```

```bash
pi@pi-red:~ $ kubectl apply -f app-purple.yaml
deployment.apps/purple configured
service/purple created
pi@pi-red:~ $ kubectl label -f app-purple.yaml release=production
deployment.apps/purple labeled
service/purple labeled
pi@pi-red:~ $ kubectl delete -f app-purple.yaml
deployment.apps "purple" deleted
service "purple" deleted
pi@pi-red:~ $ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
hasher-99bbd4bb-h955v       1/1     Running   0          82m
pingpong-5859ffb968-h5qmm   1/1     Running   0          115m
purple                      1/1     Running   0          19m
redis-7b47f84cc4-975m7      1/1     Running   0          82m
registry-848598b5b6-pbkgj   1/1     Running   0          105m
rng-65d885d498-lcw97        1/1     Running   0          82m
webui-74bb6bbc59-d4sft      1/1     Running   0          82m
worker-5c6f84b477-nrh7g     1/1     Running   0          82m
yellow                      1/1     Running   0          17m
```

## Pruning¹ resources

- We can also tell `kubectl` to remove old resources

- This is done with `kubectl apply -f ... --prune`

- It will remove resources that don't exist in the YAML file(s)

- But only if they were created with `kubectl apply` in the first place

  (technically, if they have an annotation `kubectl.kubernetes.io/last-applied-configuration`)

.footnote[¹If English is not your first language: *to prune* means to remove dead or overgrown branches in a tree, to help it to grow.]

## Advantage of YAML

- Using YAML (instead of `kubectl create <kind>`) allows to be *declarative*

- The YAML describes the desired state of our cluster and applications

- YAML can be stored, versioned, archived (e.g. in git repositories)

- To change resources, change the YAML files

  (instead of using `kubectl edit`/`scale`/`label`/etc.)

- Changes can be reviewed before being applied

  (with code reviews, pull requests ...)

- Our version control system now has a full history of what we deploy

## GitOps

- This workflow is sometimes called "GitOps"

- There are tools to facilitate it, e.g. Flux, ArgoCD...

- Compares to "Infrastructure-as-Code", but for app deployments

## Actually GitOps?

There is some debate around the "true" definition of GitOps:

*My applications are defined with manifests, templates, configurations...
that are stored in source repositories with version control,
and I only make changes to my applications by changing these files,
like I would change source code.*

vs

*Same, but it's only "GitOps" if the deployment of the manifests is
full automated (as opposed to manually running commands like `kubectl apply`
or more complex scripts or tools).*

Your instructor may or may not have an opinion on the matter! 😁

## YAML in practice

- Get started with `kubectl create deployment` and `kubectl expose`

  (until you have something that works)

- Then, run these commands again, but with `-o yaml --dry-run=client`

  (to generate and save YAML manifests)

- Try to apply these manifests in a clean environment

  (e.g. a new Namespace)

- Check that everything works; tweak and iterate if needed

- Commit the YAML to a repo 💯🏆️

## "Day 2" YAML

- Don't hesitate to remove unused fields

  (e.g. `creationTimestamp: null`, most `{}` values...)

- Check your YAML with:

  [kube-score](https://github.com/zegl/kube-score) (installable with krew)

  [kube-linter](https://github.com/stackrox/kube-linter)

- Check live resources with tools like [popeye](https://popeyecli.io/)

- Remember that like all linters, they need to be configured for your needs!


## Specifying the namespace

- When creating resources from YAML manifests, the namespace is optional

- If we specify a namespace:

  - resources are created in the specified namespace

  - this is typical for things deployed only once per cluster

  - example: system components, cluster add-ons ...

- If we don't specify a namespace:

  - resources are created in the current namespace

  - this is typical for things that may be deployed multiple times

  - example: applications (production, staging, feature branches ...)

# Namespaces

- Resources like Pods, Deployments, Services... exist in *Namespaces*

- So far, we (probably) have been using the `default` Namespace

- We can create other Namespaces to organize our resources

---

## Use-cases

- Example: a "dev" cluster where each developer has their own Namespace

  (and they only have access to their own Namespace, not to other folks' Namespaces)

- Example: a cluster with one `production` and one `staging` Namespace

  (with similar applications running in each of them, but with different sizes)

- Example: a "production" cluster with one Namespace per application

  (or one Namespace per component of a bigger application)

- Example: a "production" cluster with many instances of the same application

  (e.g. SAAS application with one instance per customer)

---

## Pre-existing Namespaces

- On a freshly deployed cluster, we typically have the following four Namespaces:

  - `default` (initial Namespace for our applications; also holds the `kubernetes` Service)

  - `kube-system` (for the control plane)

  - `kube-public` (contains one ConfigMap for cluster discovery)

  - `kube-node-lease` (in Kubernetes 1.14 and later; contains Lease objects)

- Over time, we will almost certainly create more Namespaces!

---

## Creating a Namespace

- Let's see two ways to create a Namespace!

.lab[

- First, with `kubectl create namespace`:
  ```bash
	pi@pi-red:~ $ kubectl create namespace blue
	namespace/blue created
  ```

- Then, with a YAML snippet:
  ```bash
	pi@pi-red:~ $ cat namespace.yml
	apiVersion: v1
	kind: Namespace
	metadata:
	name: green

	pi@pi-red:~ $ kubectl apply -f namespace.yml
	namespace/green created
  ```

## Using namespaces

- We can pass a `-n` or `--namespace` flag to most `kubectl` commands

.lab[

- Create a Deployment in the `blue` Namespace:
  ```bash
	pi@pi-red:~ $ kubectl create deployment purple --image jpetazzo/color --namespace blue
	deployment.apps/purple created
  ```

- Check the Pods that were just created:
  ```bash
	pi@pi-red:~ $ kubectl get pods --all-namespaces
	NAMESPACE      NAME                             READY   STATUS    RESTARTS       AGE
	blue           purple-65bb9bc655-5qw44          1/1     Running   0              16s
	default        hasher-99bbd4bb-h955v            1/1     Running   0              135m
	default        pingpong-5859ffb968-h5qmm        1/1     Running   0              168m
	default        purple                           1/1     Running   0              72m
	default        redis-7b47f84cc4-975m7           1/1     Running   0              135m
	default        registry-848598b5b6-pbkgj        1/1     Running   0              158m
	default        rng-65d885d498-lcw97             1/1     Running   0              135m
	default        webui-74bb6bbc59-d4sft           1/1     Running   0              135m
	default        worker-5c6f84b477-nrh7g          1/1     Running   0              135m
	default        yellow                           1/1     Running   0              71m
	kube-flannel   kube-flannel-ds-9m6jg            1/1     Running   0              171m
	kube-flannel   kube-flannel-ds-f6prh            1/1     Running   1 (170m ago)   171m
	kube-system    coredns-674b8bbfcf-5mbnq         1/1     Running   0              172m
	kube-system    coredns-674b8bbfcf-gzchj         1/1     Running   0              172m
	kube-system    etcd-pi-red                      1/1     Running   13             172m
	kube-system    kube-apiserver-pi-red            1/1     Running   0              172m
	kube-system    kube-controller-manager-pi-red   1/1     Running   0              172m
	kube-system    kube-proxy-bsmnk                 1/1     Running   0              171m
	kube-system    kube-proxy-rs5g9                 1/1     Running   0              172m
	kube-system    kube-scheduler-pi-red            1/1     Running   0              172m

	pi@pi-red:~ $ kubectl get pods --all-namespaces --selector app=purple
	NAMESPACE   NAME                      READY   STATUS    RESTARTS   AGE
	blue        purple-65bb9bc655-5qw44   1/1     Running   0          27s
  ```

## Switching the active Namespace

- We can change the "active" Namespace

- This is useful if we're going to work in a given Namespace for a while

  - it is easier than passing `--namespace ...` all the time

  - it helps to avoid mistakes
    <br/>
    (e.g.: `kubectl delete -f foo.yaml` whoops wrong Namespace!)

- We're going to see ~~one~~ ~~two~~ three different methods to switch namespaces!

---

## Method 1 (kubens/kns)

- To switch to the `blue` Namespace, run:
  ```bash
  kubens blue
  ```

- `kubens` is sometimes renamed or aliased to `kns`

  (even less keystrokes!)

- `kubens -` switches back to the previous Namespace

- Pros: probably the easiest method out there

- Cons: `kubens` is an extra tool that you need to install

---

## Method 2 (edit kubeconfig)

- Edit `~/.kube/config`

- There should be a `namespace:` field somewhere

  - except if we haven't changed Namespace yet!

  - in that case, change Namespace at least once using another method

- We can just edit that file, and `kubectl` will use the new Namespace from now on

- Pros: kind of easy; doesn't require extra tools

- Cons: there can be multiple `namespace:` fields in that file; difficult to automate

---

## Method 3 (kubectl config)

- To switch to the `blue` Namespace, run:
  ```bash
	pi@pi-red:~ $ kubectl config set-context --current --namespace=blue
	Context "kubernetes-admin@kubernetes" modified.
  ```

- This automatically edits the kubeconfig file

- This is exactly what `kubens` does behind the scenes!

- Pros: always works (as long as we have `kubectl`)

- Cons: long and complicated to type

  (but it's a good exercise for our fingers, maybe?)

## What are contexts?

- Context = cluster + user + namespace

- Useful to quickly switch between multiple clusters

  (e.g. dev, prod, or different applications, different customers...)

- Also useful to quickly switch between identities

  (e.g. developer with "regular" access vs. cluster-admin)

- Switch context with `kubectl config set-context` or `kubectx` / `kctx`

- It is also possible to switch the kubeconfig file altogether

  (by specifying `--kubeconfig` or setting the `KUBECONFIG` environment variable)


## What's in a context

- NAME is an arbitrary string to identify the context

- CLUSTER is a reference to a cluster

  (i.e. API endpoint URL, and optional certificate)

- AUTHINFO is a reference to the authentication information to use

  (i.e. a TLS client certificate, token, or otherwise)

- NAMESPACE is the namespace

  (empty string = `default`)

---

## Namespaces, Services, and DNS

- When a Service is created, a record is added to the Kubernetes DNS

- For instance, for service `auth` in domain `staging`, this is typically:

  `auth.staging.svc.cluster.local`

- By default, Pods are configured to resolve names in their Namespace's domain

- For instance, a Pod in Namespace `staging` will have the following "search list":

  `search staging.svc.cluster.local svc.cluster.local cluster.local`

---

## Pods connecting to Services

- Let's assume that we are in Namespace `staging`

- ... and there is a Service named `auth`

- ... and we have code running in a Pod in that same Namespace

- Our code can:

  - connect to Service `auth` in the same Namespace with `http://auth/`

  - connect to Service `auth` in another Namespace (e.g. `prod`) with `http://auth.prod`

  - ... or `http://auth.prod.svc` or `http://auth.prod.svc.cluster.local`

---

## Deploying multiple instances of a stack

If all the containers in a given stack use DNS for service discovery,
that stack can be deployed identically in multiple Namespaces.

Each copy of the stack will communicate with the services belonging
to the stack's Namespace.

Example: we can deploy multiple copies of DockerCoins, one per
Namespace, without changing a single line of code in DockerCoins,
and even without changing a single line of code in our YAML manifests!

This is similar to what can be achieved e.g. with Docker Compose
(but with Docker Compose, each stack is deployed in a Docker "network"
instead of a Kubernetes Namespace).

---

## Namespaces and isolation

- Namespaces *do not* provide isolation

- By default, Pods in e.g. `prod` and `staging` Namespaces can communicate

- Actual isolation is implemented with *network policies*

- Network policies are resources (like deployments, services, namespaces...)

- Network policies specify which flows are allowed:

  - between pods

  - from pods to the outside world

  - and vice-versa

---

##  `kubens` and `kubectx`

- These tools are available from https://github.com/ahmetb/kubectx

- They were initially simple shell scripts, and are now full-fledged Go programs

- On our clusters, they are installed as `kns` and `kctx`

  (for brevity and to avoid completion clashes between `kubectx` and `kubectl`)

# Setting up Kubernetes

- Kubernetes is made of many components that require careful configuration

- Secure operation typically requires TLS certificates and a local CA

  (certificate authority)

- Setting up everything manually is possible, but rarely done

  (except for learning purposes)

- Let's do a quick overview of available options!

---

## Local development

- Are you writing code that will eventually run on Kubernetes?

- Then it's a good idea to have a development cluster!

- Instead of shipping containers images, we can test them on Kubernetes

- Extremely useful when authoring or testing Kubernetes-specific objects

  (ConfigMaps, Secrets, StatefulSets, Jobs, RBAC, etc.)

- Extremely convenient to quickly test/check what a particular thing looks like

  (e.g. what are the fields a Deployment spec?)

---

## One-node clusters

- It's perfectly fine to work with a cluster that has only one node

- It simplifies a lot of things:

  - pod networking doesn't even need CNI plugins, overlay networks, etc.

  - these clusters can be fully contained (no pun intended) in an easy-to-ship VM or container image

  - some of the security aspects may be simplified (different threat model)

  - images can be built directly on the node (we don't need to ship them with a registry)

- Examples: Docker Desktop, k3d, KinD, MicroK8s, Minikube

  (some of these also support clusters with multiple nodes)

---

## Managed clusters ("Turnkey Solutions")

- Many cloud providers and hosting providers offer "managed Kubernetes"

- The deployment and maintenance of the *control plane* is entirely managed by the provider

  (ideally, clusters can be spun up automatically through an API, CLI, or web interface)

- Given the complexity of Kubernetes, this approach is *strongly recommended*

  (at least for your first production clusters)

- After working for a while with Kubernetes, you will be better equipped to decide:

  - whether to operate it yourself or use a managed offering

  - which offering or which distribution works best for you and your needs

---

## Managed ≠ managed

- Managed Kubernetes ≠ managed hosting

- Managed hosting typically means that the hosting provider takes care of:

  - installation, upgrades, time-sensitive security patches, backups

  - logging and metrics collection

  - setting up supervision, alerts, and on-call rotation

- Managed Kubernetes typically means that the hosting provider takes care of:

  - installation

  - maybe upgrades (kind of; you typically need to initiate/coordinate them)

  - and that's it!

---

## "Managed" Kubernetes

- "Managed Kubernetes" gives us the equivalent of a raw VM

- We still need to add a lot of things to make it production-ready

  (upgrades, logging, supervision...)

- We also need some almost-essential components that don't always come out of the box

  - ingress controller

  - network policy controller

  - storage class...

📽️[How to make Kubernetes ryhme with production readiness](https://www.youtube.com/watch?v=6G4v-ZE6OHI
)

---

## Observability

- Logging, metrics, traces...

- Pick a solution (self-hosted, as-a-service?)

- Configure control plane, nodes, various components

- Set up dashboards, track important metrics

  (e.g. on AWS, track inter-AZ and external traffic per app to avoid $$$ surprises)

- Set up supervision, on-call notifications, on-call rotation

---

## Backups

- Full machine backups of the nodes?

  (not very effective)

- Backup of control plane data?

  (important; it's not always possible to obtain etcd backups)

- Backup of persistent volumes?

  (good idea; but not always effective)

- App-level backups, e.g. database dumps, log-shipping?

  (more effective and reliable; more work depending on the app and database)

---

## Upgrades

- Control plane

  *typically automated by the provider; but might cause breakage*

- Nodes

  *best case scenario: can be done in-place; otherwise: requires provisioning new nodes*

- Additional components (ingress controller, operators, etc.)

  *depends wildly of the components!*

---

## It's dangerous to go alone!

Don't hesitate to hire help before going to production with your first K8S app!

---

## Node management

- Most "Turnkey Solutions" offer fully managed control planes

  (including control plane upgrades, sometimes done automatically)

- However, with most providers, we still need to take care of *nodes*

  (provisioning, upgrading, scaling the nodes)

- Example with Amazon EKS ["managed node groups"](https://docs.aws.amazon.com/eks/latest/userguide/managed-node-groups.html):

  *...when bugs or issues are reported [...] you're responsible for deploying these patched AMI versions to your managed node groups.*

---

## Managed clusters differences

- Most providers let you pick which Kubernetes version you want

  - some providers offer up-to-date versions

  - others lag significantly (sometimes by 2 or 3 minor versions)

- Some providers offer multiple networking or storage options

- Others will only support one, tied to their infrastructure

  (changing that is in theory possible, but might be complex or unsupported)

- Some providers let you configure or customize the control plane

  (generally through Kubernetes "feature gates")

---

## Choosing a provider

- Pricing models differ from one provider to another

  - nodes are generally charged at their usual price

  - control plane may be free or incur a small nominal fee

- Beyond pricing, there are *huge* differences in features between providers

- The "major" providers are not always the best ones!

- See [this page](https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/) for a list of available providers

---

## Kubernetes distributions and installers

- If you want to run Kubernetes yourselves, there are many options

  (free, commercial, proprietary, open source ...)

- Some of them are installers, while some are complete platforms

- Some of them leverage other well-known deployment tools

  (like Puppet, Terraform ...)

- There are too many options to list them all

  (check [this page](https://kubernetes.io/partners/#iframe-landscape-conformance) for an overview!)

---

## kubeadm

- kubeadm is a tool part of Kubernetes to facilitate cluster setup

- Many other installers and distributions use it (but not all of them)

- It can also be used by itself

- Excellent starting point to install Kubernetes on your own machines

  (virtual, physical, it doesn't matter)

- It even supports highly available control planes, or "multi-master"

  (this is more complex, though, because it introduces the need for an API load balancer)

---

## Manual setup

- The resources below are mainly for educational purposes!

- [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way) by Kelsey Hightower

  *step by step guide to install Kubernetes on GCP, with certificates, HA...*

- [Deep Dive into Kubernetes Internals for Builders and Operators](https://www.youtube.com/watch?v=3KtEAa7_duA)

  *conference talk setting up a simplified Kubernetes cluster - no security or HA*

- 🇫🇷[Démystifions les composants internes de Kubernetes](https://www.youtube.com/watch?v=OCMNA0dSAzc)

  *improved version of the previous one, with certs and recent k8s versions*

---

## About our training clusters

- How did we set up these Kubernetes clusters that we're using?

--

- We used `kubeadm` on freshly installed VM instances running Ubuntu LTS

    1. Install Docker

    2. Install Kubernetes packages

    3. Run `kubeadm init` on the first node (it deploys the control plane on that node)

    4. Set up  Weave (the overlay network) with a single `kubectl apply` command

    5. Run `kubeadm join` on the other nodes (with the token produced by `kubeadm init`)

    6. Copy the configuration file generated by `kubeadm init`

- Check the [prepare VMs README](https://@@GITREPO@@/blob/master/prepare-vms/README.md) for more details

---

## `kubeadm` "drawbacks"

- Doesn't set up Docker or any other container engine

  (this is by design, to give us choice)

- Doesn't set up the overlay network

  (this is also by design, for the same reasons)

- HA control plane requires [some extra steps](https://kubernetes.io/docs/setup/independent/high-availability/)

- Note that HA control plane also requires setting up a specific API load balancer

  (which is beyond the scope of kubeadm)










