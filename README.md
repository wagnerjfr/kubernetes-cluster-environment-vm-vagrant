# `kubernetes-cluster-vm-vagrant`
Manually install Kubernetes cluster on Oracle Linux 7 VirtualBox on your laptop :computer: in minutes and with few commands with Vagrant

## Prerequisites
- Install [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Install [Vagrant](https://vagrantup.com/)
- An account in [Oracle Container Registry](https://container-registry.oracle.com/)

## References: 
- https://blogs.oracle.com/scoter/oracle-vm-virtualbox-get-kubernetes-cluster-running-in-minutes
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## 1. Creating the k8s cluster
We are going to build a cluster with 3 nodes: 1 master and 2 workers.

### 1.1. Clone the GitHub repository for Kubernetes Vagrant boxes
```
$ git clone https://github.com/oracle/vagrant-boxes

$ cd vagrant-boxes/Kubernetes
```

### 1.2. Build Kubernetes master server
Start a new VirtualBox named `master` using vagrant and `ssh` into it:
```
$ vagrant up master; vagrant ssh master
```
Within the `master` guest, run the command below to run a shell with root privileges as `root`:
```
$ sudo -s;
```
:warning: If you face this issue that outputs in the terminal,
```console
    master: [ERROR] Please allow iptables default FORWARD rule to ACCEPT
    master:         the way to do it:
    master:         # /sbin/iptables -P FORWARD ACCEPT
    master: /tmp/vagrant-shell: Worker node ready
  
Welcome to Oracle Linux Server release 7.5 (GNU/Linux 4.14.35-1818.0.9.el7uek.x86_64)
```
run the command below as the terminal says:
```
$ /sbin/iptables -P FORWARD ACCEPT
```
You will be asked to log in to the [Oracle Container Registry](https://container-registry.oracle.com/) when you run the command below:
```
$ /vagrant/scripts/kubeadm-setup-master.sh
```
After some minutes you should have something like:
```console
[root@master vagrant]# /vagrant/scripts/kubeadm-setup-master.sh
/vagrant/scripts/kubeadm-setup-master.sh: Login to container-registry.oracle.com
Username: wagner.franchin@oracle.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
/vagrant/scripts/kubeadm-setup-master.sh: Setup Master node -- be patient!
/vagrant/scripts/kubeadm-setup-master.sh: Copying admin.conf for vagrant user
/vagrant/scripts/kubeadm-setup-master.sh: Copying admin.conf into host directory
/vagrant/scripts/kubeadm-setup-master.sh: Saving token for worker nodes
I1213 23:15:37.969880    3427 version.go:236] remote version is much newer: v1.17.0; falling back to: stable-1.12
/vagrant/scripts/kubeadm-setup-master.sh: Master node ready, run
	/vagrant/scripts/kubeadm-setup-worker.sh
on the worker nodes
[root@master vagrant]# 
```
### 1.3. Build Kubernetes worker server
On a new terminal, start a new VirtualBox named `worker1` using vagrant and `ssh` into it:
```
$ vagrant up worker1; vagrant ssh worker1
```
Within the `worker1` guest, run the command below to run a shell with root privileges as `root`:
```
$ sudo -s;
```
:warning: If you face this issue that outputs in the terminal,
```console
    worker1: [ERROR] Please allow iptables default FORWARD rule to ACCEPT
    worker1:         the way to do it:
    worker1:         # /sbin/iptables -P FORWARD ACCEPT
    worker1: /tmp/vagrant-shell: Worker node ready
  
Welcome to Oracle Linux Server release 7.5 (GNU/Linux 4.14.35-1818.0.9.el7uek.x86_64)
```
run the command below as the terminal says:
```
$ /sbin/iptables -P FORWARD ACCEPT
```
You will be asked to log in to the [Oracle Container Registry](https://container-registry.oracle.com/) when you run the command below:
```
$ /vagrant/scripts/kubeadm-setup-worker.sh
```
After some minutes you should have something like:
```console
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.

/vagrant/scripts/kubeadm-setup-worker.sh: Worker node ready
[root@worker1 vagrant]# 
```
:arrow_heading_up: ***repeat the same for worker2 server*** :arrow_heading_up:

### 1.4. Checking the k8s cluster
In `master` machine, run as `vagrant` user (and **NOT** `root` user):
```
$ kubectl get nodes
$ kubectl cluster-info
```
Output:
```console
[vagrant@master ~]$ kubectl get nodes
NAME                 STATUS   ROLES    AGE     VERSION
master.vagrant.vm    Ready    master   60m     v1.12.7+1.2.3.el7
worker1.vagrant.vm   Ready    <none>   2m43s   v1.12.7+1.2.3.el7
worker2.vagrant.vm   Ready    <none>   77s     v1.12.7+1.2.3.el7

[vagrant@master ~]$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.100:6443
KubeDNS is running at https://192.168.99.100:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

## 2. Config host machine to access local k8s
Within the `master` guest, run the command below to run a shell with root privileges as `root`:
```
$ sudo -s;
```
`cat` this file:
```
$ cat /etc/kubernetes/admin.conf
```
Select all the content of the file, and copy it.

In the host machine, create a new file, paste the content on it and save.

My file is located in `/home/wfranchi/k8s.conf`. Then you need to export it, run:
```
$ export KUBECONFIG=~/k8s.conf
```
After this, you are able to access the k8s cluster using `kubectl` from your host computer.

Try the folloing commands:
```
$ kubectl config view

$ kubectl get pods --all-namespaces
```
Output:
```console
wfranchi@computer:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.99.100:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

wfranchi@computer:~$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   coredns-b7df4d4c4-kkks7                     1/1     Running   0          93m
kube-system   coredns-b7df4d4c4-xvtkp                     1/1     Running   0          93m
kube-system   etcd-master.vagrant.vm                      1/1     Running   0          92m
kube-system   kube-apiserver-master.vagrant.vm            1/1     Running   0          93m
kube-system   kube-controller-manager-master.vagrant.vm   1/1     Running   0          93m
kube-system   kube-flannel-ds-fjsp5                       1/1     Running   0          34m
kube-system   kube-flannel-ds-llds8                       1/1     Running   0          36m
kube-system   kube-flannel-ds-r6hbm                       1/1     Running   0          93m
kube-system   kube-proxy-4dxxr                            1/1     Running   0          93m
kube-system   kube-proxy-9f4f8                            1/1     Running   0          34m
kube-system   kube-proxy-k6fkm                            1/1     Running   0          36m
kube-system   kube-scheduler-master.vagrant.vm            1/1     Running   0          93m
kube-system   kubernetes-dashboard-669df9cb5d-x5hgk       1/1     Running   0          93m

```
## 3. Deploying nginx to the new cluster as an example
This section just demostrates how to deploy nginx in the new k8s, run them separately:
```
$ kubectl create namespace test

$ kubectl create deployment nginx --image=nginx --namespace test

$ kubectl get pods --namespace test

$ kubectl get deployment nginx --namespace test

$ kubectl scale --current-replicas=1 --replicas=10 deployment/nginx --namespace test

$ kubectl get pods --namespace test

$ kubectl delete deployment nginx --namespace test

$ kubectl delete namespace test
```
Output:
```console
wfranchi@computer:~$ kubectl create namespace test
namespace/test created

wfranchi@computer:~$ kubectl create deployment nginx --image=nginx --namespace test
deployment.apps/nginx created

wfranchi@computer:~$ kubectl get pods --namespace test
NAME                    READY   STATUS              RESTARTS   AGE
nginx-55bd7c9fd-8hm29   0/1     ContainerCreating   0          17s

wfranchi@computer:~$ kubectl get deployment nginx --namespace test
NAME    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx   1         1         1            1           39s

wfranchi@computer:~$ kubectl scale --current-replicas=1 --replicas=10 deployment/nginx --namespace test
deployment.extensions/nginx scaled

wfranchi@computer:~$ kubectl get pods --namespace test
NAME                    READY   STATUS              RESTARTS   AGE
nginx-55bd7c9fd-25xk9   1/1     Running             0          29s
nginx-55bd7c9fd-428vm   0/1     ContainerCreating   0          29s
nginx-55bd7c9fd-4wrjh   0/1     ContainerCreating   0          29s
nginx-55bd7c9fd-8hm29   1/1     Running             0          83s
nginx-55bd7c9fd-cn799   1/1     Running             0          29s
nginx-55bd7c9fd-fdxpv   0/1     ContainerCreating   0          29s
nginx-55bd7c9fd-nnlcp   1/1     Running             0          29s
nginx-55bd7c9fd-nzkkq   1/1     Running             0          29s
nginx-55bd7c9fd-w4pg6   0/1     ContainerCreating   0          30s
nginx-55bd7c9fd-zcr4d   0/1     ContainerCreating   0          29s

wfranchi@computer:~$ kubectl get pods --namespace test
NAME                    READY   STATUS    RESTARTS   AGE
nginx-55bd7c9fd-25xk9   1/1     Running   0          47s
nginx-55bd7c9fd-428vm   1/1     Running   0          47s
nginx-55bd7c9fd-4wrjh   1/1     Running   0          47s
nginx-55bd7c9fd-8hm29   1/1     Running   0          101s
nginx-55bd7c9fd-cn799   1/1     Running   0          47s
nginx-55bd7c9fd-fdxpv   1/1     Running   0          47s
nginx-55bd7c9fd-nnlcp   1/1     Running   0          47s
nginx-55bd7c9fd-nzkkq   1/1     Running   0          47s
nginx-55bd7c9fd-w4pg6   1/1     Running   0          48s
nginx-55bd7c9fd-zcr4d   1/1     Running   0          47s

wfranchi@computer:~$ kubectl delete deployment nginx --namespace test
deployment.extensions "nginx" deleted

wfranchi@computer:~$ kubectl delete namespace test
namespace "test" deleted
```

## 4. Clean up
To destroy the VMs, on the host machine, `cd` into the folder you cloned before `vagrant-boxes/Kubernetes` and run:
```
$ vagrant destroy
```
Output:
```console
wfranchi@computer:~/vagrant-boxes/Kubernetes$ vagrant destroy
    worker2: Are you sure you want to destroy the 'worker2' VM? [y/N] y
==> worker2: Forcing shutdown of VM...
==> worker2: Destroying VM and associated drives...
    worker1: Are you sure you want to destroy the 'worker1' VM? [y/N] y
==> worker1: Forcing shutdown of VM...
==> worker1: Destroying VM and associated drives...
    master: Are you sure you want to destroy the 'master' VM? [y/N] y
==> master: Forcing shutdown of VM...
==> master: Destroying VM and associated drives...
```
