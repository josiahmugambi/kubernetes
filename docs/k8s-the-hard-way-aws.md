# Kubernetes The Hard Way, on AWS

The idea is to work through [this adaptation for AWS by Prabhat Sharma](https://github.com/prabhatsharma/kubernetes-the-hard-way-aws) and learn as much as possible along the way. This is also a way to polish up on my AWS knowledge as I'm more comfortable on GCP's CLI.


#### Cluster Details:
- Kubernetes - [v1.21.1 (12 May 2021)](https://kubernetes.io/releases/)
- containerd -
- cri-tools -
- CNI-plugins -
- etcd -
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
** to be continued from here **




