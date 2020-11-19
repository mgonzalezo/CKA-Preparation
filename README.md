# CKA-Preparation
Repository with some reference information about the CKA exam, tips &amp; tricks and Reference URLs. For CKAD Tips, you can refer to [this link](https://github.com/mgonzalezo/CKAD_Preparation). 

### Note 

* There are so many forums and guidelines about this exam, so I will try to explain the preparation I went through, some details I found while taking this exam and some tips I can give to those who are planning to take this 2020 CKA exam.

* Please notice that all the views and opinions expressed here are entirely personal and do not necessarily reflect the official documentation or position by CNCF.


# 1. EXAM DETAILS

CKA is a hands-on exam provided by CNCF (Cloud Native Computing Foundation). You can get all the general (an official) details of this exam here: [link to CKA official documentation!](https://www.cncf.io/certification/cka/). 

According to Linux Foundation: 

"A certified K8s administrator can work proficiently to design, install, configure, and manage a Kubernetes production-grade cluster. They will have an understanding of key concepts such as Kubernetes networking, storage, security, maintenance, logging and monitoring, application lifecycle, troubleshooting, API object primitives and the ability to establish basic use-cases for end users."

Since September 1st, 2020 CNCF made some changes on the CKA program. I will highlight the key changes you need be aware of:

1. CKA 2020 Domain & Competencies:

    - Cluster Architecture, Installation & Configuration – 25%
  
      - [ ] Manage role based access control (RBAC)
      - [ ] Use Kubeadm to install a basic cluster
      - [x] Manage a highly-available Kubernetes cluster
      - [ ] Provision underlying infrastructure to deploy a Kubernetes cluster
      - [x] Perform a version upgrade on a Kubernetes cluster using Kubeadm
      - [x] Implement etcd backup and restore
      
    - Services & Networking – 20%
    
      - [x] Understand host networking configuration on the cluster nodes
      - [x] Understand connectivity between Pods
      - [x] Understand ClusterIP, NodePort, LoadBalancer service types and endpoints
      - [x] Know how to use Ingress controllers and Ingress resources
      - [ ] Know how to configure and use CoreDNS
      - [ ] Choose an appropriate container network interface plugin
      
    - Workloads & Scheduling – 15%
    
      - [X] Understand deployments and how to perform rolling update and rollbacks
      - [X] Use ConfigMaps and Secrets to configure applications
      - [X] Know how to scale applications
      - [ ] Understand the primitives used to create robust, self-healing, application deployments
      - [ ] Understand how resource limits can affect Pod scheduling
      - [X] Awareness of manifest management and common templating tools
      
    - Storage – 10%
    
      - [X] Understand storage classes, persistent volumes
      - [X] Understand volume mode, access modes and reclaim policies for volumes
      - [X] Understand persistent volume claims primitive
      - [X] Know how to configure applications with persistent storage
    
    - Troubleshooting – 30%
     
      - [X] Evaluate cluster and node logging
      - [X] Understand how to monitor applications
      - [X] Manage container stdout & stderr logs
      - [ ] Troubleshoot application failure
      - [ ] Troubleshoot cluster component failure
      - [X] Troubleshoot networking
      
 2. Duration of exam:
 
    - From 3 to 2 hours.
 
 3. Software Version:
  
    - CKA exam now uses latest Kubernetes v1.9. For more details on this new version (e.g. release features), please refer to this URL: [Link to kubernetes 1.19](https://kubernetes.io/blog/2020/08/26/kubernetes-release-1.19-accentuate-the-paw-sitive/)

 4. Passing Score:
 
    - Last, but maybe the most relevant change, the passing score changed from 74% to 66%, same as CKAD program.
    
Note: In the CKA domain and competencies item, I marked the topics I saw during the exam. Please notice that, you must know these topics before taking this exam, as they are correlated.

# 2. HOT TOPICS

In this section, I'm going to go through some additional key topics I focused while preparing for this exam. Further information as theory can be found in official Kubernetes documentation. I'm skipping basic topics such us: Pod creation, Services creation, etc.


## STATIC POD:

Design:

To find config path:

1) Execute: ps -ef | grep kubelet
2) search for --config
3) grep -i static /var/lib/kubelet/config.yaml

USUALLY, this is the podStaticPath: /etc/kubernetes/manifests

Kubelet can create pods from Kube API or static pod config

How to create static pod within an specific Node?

- ssh node0X
- systemctl kubelet status (always good to check this)
- cd /var/lib/kubelet
- vi config.yaml
- Find the "staticPodPath" entry and verify if the /etc/kubernetes/manifests is there.
- If the folder is not there, create one.
- Then create the [pod-name].yaml file with the required information.
- verify the existance of the pod by running: docker -ps |grep [pod name]
- Go back to master node.
- Run kubectl get nodes to check new pod.

## MONITORING:

git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
cd kubernetes-metrics-server
kubectl create -f .

Logging:

Docker:
docker run -d kodekloud/event-simulator
docker log -f ecf (docker name)

Kubernetes:

kubectl logs -f POD_NAME container_name

Filter values using kubectl explain:

- kubectl explain pods --recursive | grep envFrom -A3


## SECRETS:

Convert data information into base64 form:

- echo -n 'mysql' | base64

Get Secret description:
- kubectl describe secrets

To get Secret credentials in hashed form:
- kubectl get secret [secret-name] -o yaml

To decode hashed credentials:

- echo -n '!"$%&/' | base64 --decode

Call a secret within a pod:

#INJECT SECRET AS ENVIRONMENT:

```
apiVersion: v1
kind: pod
metadata:
  name: sss
  labels:
    name: sss
spec:
  containers:
  - name: sss
    image: ddd
    ports:
	  - containerPort: 8080
	envFrom:
	  - secretRef:
	        name: app-secret
```

### Inject Secrets as environment variables:

```
apiVersion: v1
kind: pod
metadata:
  name: sss
  labels:
    name: sss
spec:
  containers:
  - name: sss
    image: ddd
    ports:
	  - containerPort: 8080
	env:
	  - name: DB_password
	    valueFrom:
		  secretKeyRef:
	        name: app-secret
			key: DB_password
```
### Injetc secrets as Volumes:

```
volumes:
- name: app-secret-volume
  secret:
    secretName: app-secret  
```
## OS UPGRADES:

General commands:

- kubectl drain [node name] : Makes a node unschedulable and terminates the node.
- kubectl cordon [node name] : Only makes the node unschedulable.
- kubectl uncordon [node name]: Makes the node schedulable again.

Show Nodes version:

- kubectl get nodes

Show version (short result)

- kubectl version --short

Show what is the latest version to upgrade:

- kubeadm upgrade plan

### Steps to start node upgrade:

1) drain existing node:
   kubectl drain master

2) kubectl get nodes

3) kubeadm version

4) apt install kubeadm=1.18.0-00

5) kubeadm upgrade apply v1.18.0

6) kubeadm version --short

7) apt install kubelet=1.18.0-00

8) kubectl get nodes

9) kubectl uncordon master

10) Apply same changes for worker nodes.

11) kubectl drain node0X

12) ssh nodexx

13) apt install kubeadm=1.18.0-00

14) kubeadm upgrade node #small process to upgrade worker node.

15) apt install kubelet=1.18.0-00

16) logout

17) kubectl get nodes

18) kubectl uncordon node01
	
	ALL UPGRADED
	
## ETCD

### ETCD Cluster
etcdctl is a command line client for etcd.

- sstores information about the state of the cluster.

export ETCDCTL_API=3

etcdctl snapshot save -h
etcdctl snapshot restore -h

PROCEDURES:

- Find the etcd version:

kubectl -n kube-system get pod
kubectl -n kube-system describe etcd-master

- ETCD CA certificate
- ETCD Server certificate path
- ETCD Server Key
- Endpoint

At what address do you reach the ETCD cluster from your master/controlplane node?

- Use the command kubectl describe pod etcd-controlplane -n kube-system and 
- Look for --listen-client-urls

### ETCD- Initial Configuration:
```
etcd
      --advertise-client-urls=https://172.17.0.30:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://172.17.0.30:2380
      --initial-cluster=controlplane=https://172.17.0.30:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.30:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://172.17.0.30:2380
      --name=controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```
### To backup ETCD:

etcd.service
--data-dir=/var/lib/etcd
```
ETCDL_API=3 etcdl \
        snapshot save snapshot.db \
		--endpoints=https://127.0.0.1:2379			This is the default as ETCD is running on master node and exposed on localhost 2379.
		--cacert=/etc/etcd/ca.crt \					verify certificates of TLS-enabled secure servers using this CA bundle
		--cert=/etc/etcd/etcd-server.crt \			identify secure client using this TLS certificate file
		--key=/etc/etcd/etcd-server.key				identify secure client using this TLS key file
```

```
ETCDL_API=3 etcdl \
        snapshot status snapshot.db
```

Run the following command to verify input parameters are correct:

```
ETCDL_API=3 etcdctl member list --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379
```
EXAMPLE RESPONSE: 345c62550c6b35b2, started, controlplane, https://172.17.0.30:2380, https://172.17.0.30:2379, false

Once this is verified, we applied the etcd backup procedure:

```
ETCDL_API=3 etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/snapshot-pre-boot.db
```
For restore procedure:

1) You can run this command if you forget the right syntax:

```
ETCDL_API=3 etcdl snapshot restore -h
```
2) Execute the following command

```
ETCDL_API=3 etcdctl snapshot restore --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 --data-dir="/var/lib/etcd-from-backup" --initial-cluster="master=https://127.0.0.1:2380" --name="master" --initial-advertise-peer-urls="https://127.0.0.1:2380" --initial-cluster-token="etcd-cluster-1" /opt/snapshot-pre-boot.db
```
==>Improved Command: 
```
ETCDL_API=3 etcdctl snapshot restore --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key" --endpoints=127.0.0.1:2379 --data-dir="/var/lib/etcd-from-backup" /opt/snapshot-pre-boot.db
```
3) cd /etc/kubernetes/manifests

4) vi etcd.yaml, Edit --data-dir, and add --initial-cluster-token value (without ""), and replace mountPath and hostPat> path for the new value selected as data directory

5) Wait until new etcd container is up:
   watch "docker ps -a | grep etcd"
   

### ETCD Security configuration:

```
apiVersion: v1
kind: Pod
metadata:
  annotations:    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://172.17.0.8:2379
  creationTimestamp: null  labels:
    component: etcd    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://172.17.0.8:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://172.17.0.8:2380
    - --initial-cluster=controlplane=https://172.17.0.8:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.8:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://172.17.0.8:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    image: k8s.gcr.io/etcd:3.4.3-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /health
        port: 2381
        scheme: HTTP
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: etcd
    resources: {}
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
```

## SECURITY

### Setup basic authentication on kubernetes

- Create a file with user details locally at /tmp/users/user-details.csv

    password123,user1,u0001
    password123,user2,u0002
    password123,user3,u0003

- Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. 
The file is located at /etc/kubernetes/manifests/kube-apiserver.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```
- Modify the kube-apiserver startup options to include the basic-auth file

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```
- Create the necessary roles and role bindings for these users:

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```  

- This role binding allows "jane" to read pods in the "default" namespace.

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Once created, you may authenticate into the kube-api server using the users credentials:

```
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```


### Yaml file to configure Role and RoleBinding:

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "create"]
```
Allow access to just specific resources:

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "create"]
  resourceNames: ["blue","orange"] <==example names
```

### ClusterRole & ClusterRoleBinding:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregate-cron-tabs-edit
  labels:
    # Add these permissions to the "admin" and "edit" default roles.
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

### ClusterRole:

kubectl create clusterrolebinding root-cluster-admin-binding --clusterrole=aggregate-cron-tabs-edit --user=michelle

Validation Command:

kubectl auth can-i [verb:create,delete,list] [element] --as [user-account] --namespace [xx]


### Image Security:

```
k create secret docker-registry private-reg-cred \
--docker-username=dock_user \
--docker-password=dock_password \
--docker-server=myprivateregistry.com:5000 \
--docker-email=dock_user@myprivateregistry.com
```

## KUBECONFIG
  
kubectl config view
  
netstat -natulp | grep kube-scheduler

SECURITY_CONFIG_FILE:

Default route:

/root/.kube/config

#kubectl config view

Set a new config file:

Run the command: kubectl config --kubeconfig=/root/my-kube-config use-context [context-name]

Validate if a certain certificate is valid

```
kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig
```

## ROLES AND ROLE-BINDINGS

-Get users assigned to an specific Role:
kubectl describe rolebinding kube-proxy -n kube-system

## CERTIFICATE SIGNING REQUESTS:

Reference Link: 
https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/

Validation Commands:

kubectl auth can-i update pods --as=john --namespace=xxx

## STORAGE

### Storage Classes:

- What is the name of the Storage Class that does not support dynamic volume provisioning?
Look for the storage class name that uses no-provisioner
Refer: https://kubernetes.io/docs/concepts/storage/storage-classes/#local to read more about Local StorageClass.

- Set to WaitForFirstConsumer.
This will delay the binding and provisioning of a PersistentVolume until a Pod using the PersistentVolumeClaim is created.

StorageClass:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
controlplane $ more dev-user-deploy.yaml
```

```
Networking:
Basic Command:
ip link
ip addr
ifconfig -about
ip link show docker0
ip route show default
netstat -nlpt
```
 2379 is the port of ETCD to which all control plane components connect to. 
 2380 is only for etcd peer-to-peer connectivity. When you have multiple master nodes


## Network Plugin

ps -aux | grep kubelet

Binaries CNI files:
--cni-bin-dir=/opt/cni/bin

What is the CNI plugin configured to be used on this kubernetes cluster?
ls /etc/cni/net.d/

k logs [weave-pod] weave -n kube-system
=> Look for ip-range

What is the range for IP service in a cluster:
=> ps -ef | grep api

What type of proxy is the kube-proxy configured to use?
=> kubectl logs kube-proxy-ft6n7 -n kube-system

```
controlplane $ kubectl logs kube-proxy-ft6n7 -n kube-system
Error from server (NotFound): pods "kube-proxy-ft6n7" not found
controlplane $ k logs kube-proxy-4r4hw -n kube-system
I1110 09:37:09.025727       1 node.go:136] Successfully retrieved node IP: 172.17.0.20
I1110 09:37:09.025833       1 server_others.go:111] kube-proxy node IP is an IPv4 address (172.17.0.20), assume IPv4 operation
W1110 09:37:09.109646       1 server_others.go:579] Unknown proxy mode "", assuming iptables proxy
I1110 09:37:09.109993       1 server_others.go:186] Using iptables Proxier.
I1110 09:37:09.116314       1 server.go:650] Version: v1.19.0
I1110 09:37:09.116988       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I1110 09:37:09.119357       1 config.go:315] Starting service config controller
I1110 09:37:09.119508       1 shared_informer.go:240] Waiting for caches to sync for service config
I1110 09:37:09.119638       1 config.go:224] Starting endpoint slice config controller
I1110 09:37:09.119758       1 shared_informer.go:240] Waiting for caches to sync for endpoint slice config
I1110 09:37:09.219789       1 shared_informer.go:247] Caches are synced for service config
I1110 09:37:09.221016       1 shared_informer.go:247] Caches are synced for endpoint slice config
```

CDNS:
----
k describe cm coredns -n kube-system
CoreDNS file:

/etc/coredns/Corefile
```
.:53 {
	errors
	health
	kubernetes cluster.local in-addr.arpa {
	   pods insecure 									<==Responsible to allow POD DNS resolution!!
	   upstream
	   fallthrough in-addr.arpa ip6.arpa
	}
	prometheus :9153
	proxy . /etc/resolv.conf
	cache 30
	reload
}
```

DNS POD Name: CoreDNS
DNS Service Name: kube-dns

Where can I find the DNS setup?
- /etc/resolv.conf
nameserver x.y.z.w

It also has a default configuration, with all kubernetes domains.

Where can I find static pod name resolution?
- /etc/host

Who is responsible to set DNS setup within the pods? Kubelet

- Path to verify this configuration: /var/lib/kubelet/config.yaml

Name resolutions: For example - webservice @ default namespace

- web-service
- web-service.default
- web-service.default.svc.
- web-service.default.svc.cluster.local

commands:

nslookup
dig
host 

How to test NSLOOKUP:

Use a busybox:1.28 image:

k run busybox --image=busybox:1.28 --rm -it -- nslookup [Service or Pod domain name] > destination/path

1) For services:

- nslookup [name of the service]

2) For Pod:

- Verify /etc/coredns/Corefile file allows Pod DNS resolution (just in case!!)
- nslookup [Pod IP in syntax: e.g. for 10.10.1.1 > 10-10-1-5.{namespace}.pod] <= important to follow this pattern.

-----
COMMAND TO READ A CERTIFICATE INFORMATION


-------------
Create Kubernetes Cluster using kubeadm

1) First download and install kubeadm, kubectl and kubelet on both Master and Worker nodes:
kubeadm: the command to bootstrap the cluster.
kubelet: the component that runs on all of the machines in your cluster and does things like starting pods and containers.
kubectl: the command line util to talk to your cluster.

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

2) Initialize Control Plane Node (Master Node)

kubeadm init 

3) Generate a kubeadm join token or just copy the one you already received

=>>kubeadm token create --print-join-command


------------------
# 3. Troubleshooting Section:

Here is where the fun begins! Troubleshooting is one of most exciting part of the exam, as it tests your full understanding of a Kubernetes cluster, and the different ways we have to solve a particular issue. I have summarized all commands I use based on specific scenarios. Feel free to try them all and solve your own Kubernetes issues!

## Applications Failures

- kubectl get pod
- kubectl describe pod [pod-name]
- kubectl logs [pod-name] -f
- kubectl logs [pod-name] -f --previous
- kubectl get endpoints ${SERVICE_NAME}
- kubectl get pods --selector=name=nginx,type=frontend
- nc -z -v -w 2 [Service Name][Service-port] #To discard Network Policy issues.

## Control Plane Failures

- kubectl get nodes
- kubectl get pods
- kubectl get pods -n kube-system
- kubectl get svc -n kube-system
- service kube-apiserver status
- service kube-controller-manager status
- service kube-scheduler status

## On worker nodes:

- service kubelet status
- service kube-proxy status
- kubectl logs kube-apiserver-master -n kube-system
- sudo journalctl -u kube-apiserver
- kubectl cluster-info dump
- /var/log/kube-apiserver.log API Server, responsible for serving the API
- /var/log/kube-scheduler.log Scheduler, responsible for making scheduling decisions
- /var/log/kube-controller-manager.log Controller that manages replication controllers
- /var/log/kubelet.log Kubelet, responsible for running containers on the node
- /var/log/kube-proxy.log Kube Proxy, responsible for service load balancing
- kubectl describe node [node_name]
  "True" - node is under a disk/memory/issue and need attention.
  "unkown" - stop communication between Master and Node.
- top
- df -h
- sudo journalctl -u kubelet
- service kubelet status
- openssl x509 -in /var/lib/kubelet/[node-name].crt -text  => To check certificates validity and status.
- journalctl -u kubelet -f


## Certification issue within Worker node:

1) Go to file: 10-kubeadm.conf
	cd /etc/systemd/system/kubelet.service.d/
2) Go to KUBELET_CONFIG_ARGS file:
	Go to path: /var/lib/kubelet/config.yaml
3) Make all necessary changes
4) Reload daemon
	systemctl daemon-reload	
5) Restar Kubelet service
	systemctl restart kubelet


## Troubleshoot Wrong Kube-API information in Worker Node:

kubectl cluster-info

1) Go to Kubelet service definition file:
	cd /etc/systemd/system/kubelet.service.d/
2) Go to the path related to KUBELET_KUBECONFIG_ARGS to get the information related to the API server is stored.
	vi /etc/kubernetes/kubelet.conf
3) Do all the necessary changes to have the correct Kube-API information.
4) Reload daemon
	systemctl daemon-reload	
5) Restar Kubelet service
	systemctl restart kubelet

------

## Troubleshooting Network-issues

1.- If you cannot allocate pods:

- Check if CNI is installed:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#steps-for-the-first-control-plane-node

- Check Kube Proxy Daemon Set, Kube Proxy ConfigMaps and check Kube proxy pod logs.

Note: config.conf CM path is /var/lib/kube-proxy/config.conf

2.- If by any reason you cannot connect to a DB pod, even if everything else LOOKs OK.... 

- Check Core-DNS pod AND kube-dns Service. Check Endpoints: kubectl get ep [service-name]



# 4. EXTRAS

Something to keep in mind during the exam is that you **MUST** find ways to save time while doing the exam, the following topics try to save you some minutes while solving the different problems:


1. Path for iteration using JSON format:

  - k get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}{end}'

  - k get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu

  - k get nodes -o=jsonpath='{range .items[*]}{.status.nodeInfo.osImage}{"\n"}'

  - kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt

  - kubectl config view --kubeconfig=my-kube-config -o jsonpath='{range .items[*]}{.contexts}{"\n"}'

  - kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

  - k get pv -o=jsonpath='{range.items[*]}{.metadata.name}{"\t"}{.spec.capacity}{"\n"}}'

  A set of Persistent Volumes are available. Sort them based on their capacity and store the result in the file /opt/outputs/storage-capacity-sorted.txt

    - kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt

    - k get pv -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage --sort-by=.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt

  Query to identify the context configured for the aws-user in the my-kube-config context file and store it:

    - kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name

2. LINUX COMMAND FOR LISTING:

    * k get ns | sed "1 d" |wc -l

    * vim , then type: ":5dd" to delete upcoming 5 lines

3.- USE OF ALIASES

Every second you save is priceless, that is why I suggest the following aliases (run them as soon the exam starts, and get familiar with this syntaxis before-hand).

- alias k=kubectl
- alias kg="kubectl get"
- alias ke="kubectl explain" (super important when a typo error is blocking your way to the glory!)
- alias kd="kubectl describe"
- alias kdel="kubectl delete"
- alias ka="kubectl apply -f "
- alias kn="kubectl config set-context --current --namespace" (use this with extreme caution, changing namespaces during exam means you need to return to default ns for every single question).

# 5. CKA vs CKAD 

When it comes to comparing these two programs, my answer will always be the same: it depends. 

It depends what is your current role and how deep you want to go into Kubernetes. **Developers** might consider CKAD as a first step and then move to CKA for full knowledge. **DevOps or Infrastructure engineers** may consider CKA as a good chance to test their Linux & Docker Knowledge.

Which one is more difficult?
Personally, **I found CKA more difficult than CKAD exam**, not for the level of questions, but the way the questions were presented (and in some cases, redacted). Sometimes I found myself guessing between two or three options for one question of CKA, while for CKAD I knew from the first time what I needed to do. However, it may be different based on your background and study plan.

# 6. FINAL WORDS

Preparing for CKA program was quite challenging, but once you get the final result and realize how much you have learnt about this amazing Cloud Native world and Microservices, you won't regret it!

![Image of CKA](https://github.com/mgonzalezo/CKA-Preparation/blob/main/cka-mgonzalezo.JPG)

![Image of CKAD](https://github.com/mgonzalezo/CKA-Preparation/blob/main/ckad-mgonzalezo.JPG)

