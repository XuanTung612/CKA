# CKA
![CKA](https://img.shields.io/badge/-CKA-0690FA?style=for-the-badge&logo=kubernetes&logoColor=white)
![K8s](https://img.shields.io/badge/-kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)

An environment made as a preparation for the Certified Kubernetes Administrator exam [CKA v1.19]

---

### :pencil: Objectives:

<details>
<summary>1- Cluster Architecture, Installation & Configuration 25%</b></summary>
<p>

1. Manage role based access control (RBAC)
2. Use Kubeadm to install a basic cluster
3. Manage a highly-available Kubernetes cluster
4. Provision underlying infrastructure to deploy a Kubernetes cluster
5. Perform a version upgrade on a Kubernetes cluster using Kubeadm
6. Implement etcd backup and restore

</p>
</details>


<details>
<summary>2- Workloads & Scheduling 15%</b></summary>
<p>

1. Understand deployments and how to perform rolling update and rollbacks
2. Use ConfigMaps and Secrets to configure applications
3. Know how to scale applications
4. Understand the primitives used to create robust, self-healing, application deployments [Probes]
5. Understand how resource limits can affect Pod scheduling [resources for pods and quota for namespaces]
6. Awareness of manifest management and common templating tools

</p>
</details>

<details>
<summary>3- Services & Networking 20%</b></summary>
<p>

1. Understand host networking configuration on the cluster nodes
2. Understand connectivity between Pods
3. Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
4. Know how to use Ingress controllers and Ingress resources
5. Know how to configure and use CoreDNS
6. Choose an appropriate container network interface plugin

</p>
</details>

<details>
<summary>4- Storage 10%</b></summary>
<p>

1. Understand storage classes, persistent volumes
2. Understand volume mode, access modes and reclaim policies for volumes
3. Understand persistent volume claims primitive
4. Know how to configure applications with persistent storage

</p>
</details>

<details>
<summary>5- Troubleshooting 30%</b></summary>
<p>

1. Evaluate cluster and node logging
2. Understand how to monitor applications
3. Manage container stdout & stderr logs
4. Troubleshoot application failure
5. Troubleshoot cluster component failure
6. Troubleshoot networking

</p>
</details>

---

### Cluster components:
Kubernetes cluster consists of one or more master nodes + one or more worker nodes, the master node components are called `Control plane`.

#### Control plane components:
- **kube-api** server
- **etcd** Key value store
- **kube-scheduler** that allocates nodes for pods
- **kube-controller-manager** and there are different types of controllers like `replication controller`, `endpoints controller`, `service account controller` and `Token controller`

#### Worker nodes components:
- **Kubelet** Ensures pods are running on the nodes
- **kube-proxy** Maintains network rules on the nodes to keep `SVCs` working. 

---

### [Important kubectl commands](https://blog.heptio.com/kubectl-explain-heptioprotip-ee883992a243):

```bash
kubectl explain deploy

# Check all fields in a resource
kubectl explain <resource> --recursive # resource can be pod, deployment, ReplicaSet etc

kubectl explain deploy.spec.strategy

kubectl config -h

kubectl proxy # runs on port 8001 by default 
# use curl http://localhost:8801 -k to see a list of API groups

# NOT kubectl but useful
journalctl -u kube-apiserver

# Dry run and validate
kubectl apply -f fileName.yml --validate --dry-run=client

```
---

### :file_folder: Important Directories:
```bash
/etc/kubernetes/pki/ # Here all certs and keys are stored

/etc/kubernetes/manifests # Here all config files are located
  /etcd.yaml
  /kube-apiserver.yaml
  /kube-controller-manager.yaml
  /kube-scheduler.yaml

$HOME/.kube/config # --kubeconfig file

/var/lib/docker # ["aufs", "containers", "image", "volumes"]

/var/logs/containers # logs are stored here 
```

---

### Important Documentation page sections:

- [kubeadm check required ports](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports) 

- [Application Introspection and Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/)

---

### :bulb: Imperative usage of kubectl command:
```bash
# View api-resources 
k api-resources -h

# View only the namespaced api-resources
k api-resources --namespaced=True

# Add annotation to deployment
k annotate deploy/<name> k=v

# Create namespace 
k create ns <name>

# Create deployment and set an env variable for it
k create deploy <name> --image=<name> 
k set env deploy/<name> K=v

# Create ConfigMap from env varibales and use it in a deployment
k create cm <name> --from-literal=K=v --from-literal=K=v 
k set env deploy/<name> --from=cm/<name>

# Those key value pairs can be stored in a .env file and also be used 
k create cm <name> --from-env-file=<name>.env
k set env deploy/<name> --from=cm/<name>

# Limit the values that will be used from a configMap using --keys option
k set env deploy/<name> --from=cm/<name> --keys="key,key" 

# Set resources for deployment 
k set resources -h 
k set resources deploy/<name> --requests=cpu=200m,memory=512Mi --limits=cpu=500m,memory=1Gi

# Create HorizontalPodAutoscaler resource [HPA] for a deployment
k autoscale deploy <name> --min=<number> --max=<number> --cpu-percent=<number>
k get hpa

# Create a secret 
k create secret generic <name> --from-literal=K=v

# Delete resource 
k delete <resource> --force --grace-period=0 

# Get using labels, either --selector or -l followed by key value pairs
k get po --selector | -l k=v # Can be done for multiple labels just separate using a comman k=v,k=v etc

# Get pods with the same label across all namespaces [-A is short for --all-namespaces]
k get po -l k=v -A

# Run a pod 
k run <name> --image=<name> -o yaml --dry-run=client > <name>.yml
k apply -f <name>.yml

# Define an env variable for a pod using --env
k run <name> --image=<name> --env K=v --env K=v -o yaml --dry-run=client 


## Labelling
# Label a node
k label nodes <node-name> k=v

# Delete a lable from the node
k label nodes <node-name> k-

## Rollout commands
k rollout -h 
k rollout [history/pause/restart/resume/status/undo] deploy/<name>

# View details of a specific revision
k rollout history deploy/<name> --revision=<number>

# Scale replicas of a deployment 
k scale deploy/<name> --replicas=<number>

```

---

<details>
<summary>Cluster Maintenance</summary>
<p>

```bash
# Mark node as unusable 
kubectl drain <node>

OR 

kubectl cordon <node>

# Remove the drain restriction
kubectl uncordon <node>
```

Cordon Vs drain:
- Cordon doesn't terminate existing pods on the node but it prevents creation of any new pods on that node
- Drain terminates those pods and they get allocated to a different node

Upgrading a cluster:
```bash
kubeadm upgrade plan
kubeadm upgrade apply
```

Backup resource configuration:
1- Backup all resources 
```bash
kubectl get all -Ao yaml > all_resources.yml
```
> :bell: Implement etcd backup and restore :bell:

2- Use etcdctl to backup the etcd server
```bash
ETCD_API=3 etcdctl snapshot save snapshot.db
```

</p>
</details>


---

<details>
<summary>Security</summary>
<p>

> :bell: Create and manage TLS certificates for cluster components :bell:

1- Certificate authority [CA]
```bash
# Generate Keys
openssl genrsa -out ca.key 2048

# Certificate Signing Request 
openssl req -new -key ca.key -subj "/CN-KUBERNETES-CA" -out ca.csr

# Sign certificates 
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

2- Client certificates [admin user]
```bash
openssl genrsa -out admin.key 2048
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr 
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

3- Kube-API server 
```bash
openssl requ -new -key apiserver.key -subj "CN=/kube-apiserver" -out apiserver.csr
```

View certificate details
```
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

View etcd logs (if setup was done using kubeadm):
```
kubectl logs etcd-master
```

Approving certificate signing request:
```
kubectl certificate approve <name>
```

View `kube-config` file:
```
kubectl config view
```

Change default context:
```bash
kubectl config use-context user@cluster
# This changes default context in the file as well so now current-context: user@cluster
```

Check given access permissions with `can-i`:
```bash

kubectl auth can-i <verb> <object>

kubectl auth can-i create deployments

kubectl auth can-i delete nodes

# use can-i as another user 
kubectl auth can-i create pods --as <user-name>

# use can-i as another user and test a different namespace 
kubectl auth can-i create pods --as <user> --namespace <name>
```

</p>
</details>


---

#### JSON Path queries:
```bash
# First view the returned output from the command, you will narrow this output down
kubectl get <object> -o json 

# Filter the output
kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'

# You can query more than one item at a time
kubectl get po -o=jsonpath='{.items[*].metadata.name}{.items[*].status.capacity.cpu}'

# Prettifying the output using tabs or newlines
kubectl get po -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'

# Loops 
'{range .items[*]}
  {.metadata.name}{"\t"} {.status.capacity.cpu}{"\n"}
{end}
'

# FInal form
kubectl get nodes -o=jsonpath='{range .items[*]} {.metadata.name} {"\t"} {.status.capacity.cpu} {"\n"} {end}'

# Custom column iteration
kubectl get nodes  -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu

```

---
### Referencing Values:
#### From pod fields:

<details>
<summary>metadata</b></summary>
<p>

.name
.namespace
.uid

</p>
</details>

<details>
<summary>spec</b></summary>
<p>

.nodeName
.serviceAccountName

</p>
</details>

<details>
<summary>status</b></summary>
<p>

.hostIP
.podIP

</p>
</details>

---

#### From container resources fields:

<details>
<summary>requests</b></summary>
<p>

.cpu
.memory

</p>
</details>

<details>
<summary>limits</b></summary>
<p>

.cpu
.memory

</p>
</details>

### :diamonds: Storage: 

.pod.spec.containers.volumeMounts
  . readOny: True 

---

### :diamonds: Workloads & Scheduling:

#### :gem:  1- Understand deployments and how to perform rolling update and rollbacks:

```bash
k create deploy/<name> --image=<name> --replicas=1
k rollout status deploy/<name>

k rollout history deploy/<name>

# Manually annotate the current revision
k annotate deploy/<name> kubernetes.io/change-cause="Initial deployment"

# Annotate the upcoming revision using the specified command 
k set image deploy/<name> name=<image> --record

# Rollback help
k rollout undo -h

# Rollback to the last revision 
k rollout undo deploy/<name>

# Rollback to specific revision with the --to-revision flag
k rollout undo deploy/<name> --to-revision=<number>


```

<details>
<summary>k explain deploy.spec.strategy.rollingUpdate</summary>
<p>

```bash
.maxSurge # Max no of pods above the desired limit [For the new revision]
.maxUnavailable # Max no of pods that can be absent from the old revision during the rollout
```

</p>
</details>

#### :gem: 2- Use ConfigMaps and Secrets to configure applications
:mag: Search the docs for:
```bash
#Configmap volume
also can be reached via 
k explain pod.spec.volumes.configmap
```

#### :gem: 3- Know how to scale applications
```bash
# Scale a deployment 
k scale deploy/<name> --replicas=<number>

# Conditionally scale using hpa 
k autoscale deploy/<name> --min=<number> --max=<number> --cpu-percent=<number>
```


#### :gem:  Awareness of manifest management and common templating tools:
**pod.spec**:
There are many fields under the `containers` in the spec, the most signifacnt ones are:
1. **Scheduling**: This helps in selecting the node where the pod will be deployed to either directly through `nodeName` , `nodeSelector` which uses a lable or using `affinity` and `tolerations`

2. **SecurityContext**: defines security attributes like `runAsUser`, `capabilites`
```bash
k explain pod.spec.containers.securityContext

# There's also securityContext available at the spec level 
k explain pod.spec.securityContext
```

<details>
<summary>k explain pod.spec</summary>
<p>

```bash
.securityContext.runAsUser
```

</p>
</details>

<details>
<summary>k explain pod.spec.containers</summary>
<p>

```bash
.securityContext.capabilites
```

</p>
</details>

3. **Lifecycle** 
  . `restartPolicy`: defines action taken after the termination of a pod.
  . `terminationGracePeriodSeconds`: fine-tune the periods after which processes running in the containers of a terminating pod are killed, The grace period is the duration in seconds after the processes running in the pod are sent a termination signal and the time when the processes are forcibly halted with a kill signal.
  . `activeDeadlineSeconds` Optional duration in seconds the pod may be active on the node relative to StartTime before the system will actively try to mark it failed and kill associated containers

<details>
<summary>k explain pod.spec</summary>
<p>

```bash
.restartPolicy
.terminationGracePeriodSeconds
.activeDeadlineSeconds
```

</p>
</details>

4. **ServiceAccount**: Gives specific rights to pods using a specific `serviceAccountName`

5. Hostname & Name resolution: 
`hostAliases`: List of hosts and IPs that gets added to `/etc/hosts` file.
`dnsConfig`:  Specifies the DNS parameters of a pod and update `/etc/resolv.conf`
`hostname`: Specifies the hostname of the Pod If not specified

<details>
<summary>k explain pod.spec</summary>
<p>

```bash
.dnsConfig
.hostAliases
```

</p>
</details>


---

### :diamonds: Troubleshooting:

```bash
k top nodes # metrics-server needs to be deployed
```

#### :gem: Evaluate cluster and node logging:

```bash
# View kubelet logs 
journalctl -u kubelet

# View logs for core componentes 
cd /var/log/containers 
cat $(ls kube-proxy*) | tail -n 1
cat $(ls coredns*) | tail -n 1
cat $(ls kube-apiserver*) | tail -n 1

# View kube-system ns pods
k get po -n kube-system

k logs <name> -n kube-system

k describe po etcd-controller -n kube-system

# Switch to kube-system namespace 
k config set-context --current --namespace=kube-system
```

#### :gem: Manage container stdout & stderr logs:
```bash
# View logs from all containers with specific label in a specific ns 
k logs -l k=v --all-containers -n <ns> 


```