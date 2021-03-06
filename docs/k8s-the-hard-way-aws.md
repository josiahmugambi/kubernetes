# Kubernetes The Hard Way, on AWS

The idea is to work through [this adaptation for AWS by Prabhat Sharma](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws) and learn as much as possible along the way. This is also a way to polish up on my AWS knowledge as I'm more comfortable on GCP's CLI.


#### Cluster Details:
- Kubernetes - [v1.21.1 (12 May 2021)](https://kubernetes.io/releases/)
- containerd -
- cri-tools -
- CNI-plugins -
- etcd - v3.5.0-rc.0
- gVisor - [https://github.com/google/gvisor/releases/tag/release-20210419.0](https://github.com/google/gvisor/releases/tag/release-20210419.0)


### Prerequisites & Client Tools
This bit was straightforward and involves following the first [lab](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/01-prerequisites.md) download/upgrade the AWS CLI, configuring the default region and installing client tools.

Software versions that I used. It's important to have the versions match for kubectl.
```
[josiah:~/dev/k8s-thw-aws/take1]$ cfssl version
Version: 1.5.0
Runtime: go1.16.3

[josiah:~/dev/k8s-thw-aws/take1]$ cfssljson --version
Version: 1.5.0
Runtime: go1.16.3

[josiah:~/dev/k8s-thw-aws/take1]$ kubectl version --client
Client Version: version.Info{Major:"1", Minor:"21", GitVersion:"v1.21.1", GitCommit:"5e58841cce77d4bc13713ad2b91fa0d961e69192", GitTreeState:"clean", BuildDate:"2021-05-12T14:11:29Z", GoVersion:"go1.16.3", Compiler:"gc", Platform:"darwin/amd64"}

```

### Provisioning Compute Resources

I generally followed the [lab](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/03-compute-resources.md).

It's also important to understand the Kubernetes Networking Model. [See Kelsey's summary](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/03-compute-resources.md). I had already gone through the Kubernetes [concepts](https://kubernetes.io/docs/concepts/) while on the GCP labs.

I had to install jq [(Lightweight and flexible command-line JSON processor](https://stedolan.github.io/jq/)) for this command to work.

```
brew install jq
```

For the controllers and workers, one might need to change the instance type depending on the region. I got this error while using instance type t3.micro in us-east-1e:

```
An error occurred (Unsupported) when calling the RunInstances operation: Your requested instance type (t3.micro) is not supported in your requested Availability Zone (us-east-1e). Please retry your request by not specifying an Availability Zone or choosing us-east-1a, us-east-1b, us-east-1c, us-east-1d, us-east-1f.
```

To check instance types supported:

```
aws ec2 describe-instance-type-offerings --location-type "availability-zone" --filters Name=location,Values=us-east-1e --region us-east-1
```

I selected t2.medium 


p.s. - I'd love to know if the aws cli has a subcommand with list. Something that allows me to do the equivalent of this on GCP :

```
gcloud compute instances list

NAME          ZONE        MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP    STATUS
controller-0  us-east1-b  e2-standard-2               10.240.0.10  35.227.78.5    RUNNING
controller-1  us-east1-b  e2-standard-2               10.240.0.11  34.73.84.217   RUNNING
controller-2  us-east1-b  e2-standard-2               10.240.0.12  35.231.41.251  RUNNING

```
### Provisioning a CA and Generating TLS Certificates
[Instructions](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/04-certificate-authority.md) are straightforward to follow. Typing (vs copy and paste) allowed me to gain a better understanding of what each command does.

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
Relatively straightfoward to do. Just followed the [instructions](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/05-kubernetes-configuration-files.md). Again, typing more useful - more so when you inevitably make a typo and have to figure out where you do so.

This section creates kubeconfig files for:

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
Instructions from the [lab](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/06-data-encryption-keys.md) are straight forward.
These are then copied to each of the controller instances.

### Bootstrapping the etcd cluster
Instructions followed [here](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws/blob/master/docs/07-bootstrapping-etcd.md). Again, typing more useful - more so when you inevitably make a typo and have to figure out where you do so, and one learns something along the way.

Commands were run using tmux

Quick steps if on iTerm2 on MacOS:
- `tmux -CC`
- split screen into 3 and log into three respective instances (would be controllers for the first bit)
- on tmux session dialog, run command `set synchronize-panes on`

successfully deployed with v3.5.0-rc.0 of etcd.

### Bootstrapping the Kubernetes Control Plane
To be continued