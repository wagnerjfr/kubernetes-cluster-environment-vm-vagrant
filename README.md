# kubernetes-cluster-vm-vagrant
Manually install Kubernetes on Oracle Linux 7 Virtual Boxes on your laptop :computer: in minutes and with few commands with Vagrant

## Prerequisites
- Install [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- Install [Vagrant](https://vagrantup.com/)
- An account in [Oracle Container Registry](https://container-registry.oracle.com/)

## References: 
- https://blogs.oracle.com/scoter/oracle-vm-virtualbox-get-kubernetes-cluster-running-in-minutes
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/

## 1. Creating the k8s cluster

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

## 2. Clean up
To destroy the VMs, on the host macine, `cd` into the folder you cloned before `vagrant-boxes/Kubernetes` and run:
```
$ vagrant destroy
```
