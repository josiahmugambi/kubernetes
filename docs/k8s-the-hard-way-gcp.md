# Learn By Doing - Kubernetes The Hard Way, on GCP

The idea is to work through [Kelsey Hightower's Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)  and learn as much as possible along the way. I've found it useful to  type out each command (vs copy/paste) as forcing function for engagement with command parameters.

I followed the labs pretty much all the way.



### Prerequisites and Client Tools

Prefer to create own project and configurations under gcloud:

```
gcloud config configurations create k8s-thw
gcloud config set project k8s-thw-313711
gcloud config set account XXXX@XXXX
gcloud config set compute/region us-east1
gcloud config set compute/zone us-east1-b
```


Cloud Flare PKI tools (cfssl)
I had this installed a while ago, so upgraded via homebrew (the downside to doing it this way is that I hadn't upgraded in a while so took quite some time across a rather slow connection while on the move)

Testing to make sure all is ok:

```
[josiah:~/dev/k8s-thw-gcp]$ cfssl version
Version: 1.5.0
Runtime: go1.16.3

[josiah:~/dev/k8s-thw-gcp]$ cfssljson --version
Version: 1.5.0
Runtime: go1.16.3

[josiah:~/dev/k8s-thw-gcp]$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.0", GitCommit:"cb303e613a121a29364f75cc67d3d580833a7479", GitTreeState:"clean", BuildDate:"2021-04-08T21:16:14Z", GoVersion:"go1.16.3", Compiler:"gc", Platform:"darwin/amd64"}
```
### Provisioning Compute Resources

##### The Kubernetes Network Model

Before jumping into provisioning compute resources I took some time to refresh on concepts.

Key take aways:
- every pod gets its own IP address. In a sense treat pods like VMs
- pod to pod communication on all nodes without NAT
- agents on a node (e.g system daemons, kubeletes) can communicate with all pods on that node
- IP-per-pod model k8s IP addresses exist at the Pod scope - containers within a pod share networking namespaces including IP addresses and mac addresses -

Also reviewed what a k8s cluster consists of

##### Control Plane:

- kube-apiserver - frontend for Control Plane. Scales horizantally
- etcd - backing store for all cluster data. is HA. Careful to keep this reliable, available and secure
- kube-scheduler - watches for newly created pods with no assigned nodes, selects node to run
- kube-controller-manager - runs controller processes as a single binary (node controller, job controller, end points controller, service accounts, token)
- cloud-controller-manager - is optional. embeds cloud specific control logic. scales horizantally

##### Node Components:

Run on worker nodes
- kubelet - agent - makes sure containers are running in a pod based on PodSpecs
- kubeproxy - network proxy that runs on each node implementing part of k8s service concept. Maintains network rules
- container runtime - e.g. docker, containerd, CRI-O
- Add Ons - use k8s resources (e.g. DaemonSet, Deployment) to implmenet cluster features within kubesystem namespace
    - DNS - services DNS records for k8s services - you'd probably have this
    - webui-dashboard
    - cluster resource monitoring
    - cluster level logging



Followed the [lab instructions](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md). Successfully created the controller and worker instances:

```
[josiah:~/dev/k8s-thw-gcp]$ gcloud compute instances list
NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
controller-0  us-east1-b  e2-standard-2               10.240.0.10  35.227.78.5    RUNNING
controller-1  us-east1-b  e2-standard-2               10.240.0.11  34.73.84.217   RUNNING
controller-2  us-east1-b  e2-standard-2               10.240.0.12  35.231.41.251  RUNNING
worker-0      us-east1-b  e2-standard-2               10.240.0.20  34.74.209.18   RUNNING
worker-1      us-east1-b  e2-standard-2               10.240.0.21  34.75.106.183  RUNNING
worker-2      us-east1-b  e2-standard-2               10.240.0.22  104.196.63.65  RUNNING
```

### Provisioning a CA and Generating TLS Certificates

[Instructions](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/04-certificate-authority.md) are straightforward to follow. Typing (vs copy and paste) allowed me to gain a better understanding of what each command does.

General flow:
- create JSON file with cert data
- generate certificate against CA and profile created previously
- once all the certificates are generated, copy to worker instances and controller instances respectively

Certificates:
- admin user
- node authorizer  (for requests by kubelets)
- Controller Manager (kube-controller-manager)
- Kube Proxy Client (kube-proxy)
- Kube Scheduler (kube-scheduler)
- API Server (kube-apiserver)
    kubernetes API server is automatically assigned the kubernetes internal dns name and is tied to the first IP address from the range 10.32.0.0/24 reserved for internal cluster services when bootstrapping
- Service Account Key-Pair (service-account)

### Generating Kubernetes Configuration Files for Authentication
Relatively straightfoward to do. Just followed the [instructions](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/05-kubernetes-configuration-files.md). Again, typing more useful - more so when you inevitably make a typo and have to figure out where you do so.

This section creates kubeconfig files for:
- kubelets: worker-N.kubeconfig
- kube-proxy
- kubernetes controller manager
- kube-scheduler
- admin user

They are then distributed as follows:
- worker instances: kubelets (respective file), kube-proxy
- controller instances: kube-controller-manager, kube-scheduler, admin 

### Generating the Data Encryption Config and Key
These are used for securely storing kubernetes data and secrets.
Instructions from the [lab](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/06-data-encryption-keys.md) are straight forward.
These are then copied to each of the controller instances.

### Bootstrapping the etcd cluster
Instructions followed [here](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/07-bootstrapping-etcd.md). Again, typing more useful - more so when you inevitably make a typo and have to figure out where you do so, and one learns something along the way.

Commands were run using tmux

Quick steps if on iTerm2 on MacOS:
- `tmux -CC`
- split screen into 3 and log into three respective instances (would be controllers for the first bit)
- on tmux session dialog, run command `set synchronize-panes on`

### Bootstrapping the Kubernetes Control Plane
To be continued