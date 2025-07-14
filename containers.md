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

.lab[

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


