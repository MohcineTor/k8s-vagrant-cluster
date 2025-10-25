# Kubernetes Cluster Deployment with Kubeadm, Ansible & Vagrant

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Vagrant](https://img.shields.io/badge/Vagrant-1563FF?style=for-the-badge&logo=vagrant&logoColor=white)

---

This project provides a fully automated way to deploy a **multi-node Kubernetes cluster** (1 master + 2 workers) locally using **Kubeadm**, **Ansible**, and **Vagrant**.


## 📖 Table of Contents
- [🔍 Key Concepts](#-key-concepts)
  - [Kubeadm](#1-kubeadm)
  - [Containerd](#2-containerd)
  - [Calico CNI](#3-calico-cni)
  - [Kubelet](#4-kubelet)
- [🚀 Deployment](#-deployment)
  - [Prerequisites](#prerequisites)
  - [Project Structure](#project-structure)
  - [Setup Vagrant VMs](#setup-vagrant-vms)
  - [Run Ansible Playbook](#run-ansible-playbook)
  - [Verify Cluster](#verify-cluster)
  - [Test Cluster](#🧪-test-du-cluster)
    - [Create Nginx Deployment](#créer-un-déploiement-nginx)
    - [Verify Deployment and Services](#verify-deployment-and-services)
    - [Access Nginx via NodePort](#access-nginx-via-nodeport)
- [🛠️ Useful Commands](#-useful-commands)
- [🧹 Cleanup](#-cleanup)

---

## 🔍 Key Concepts

### 1. Kubeadm
**Definition**  
Kubeadm is a Kubernetes tool that simplifies cluster bootstrapping. It handles:  
- Initialization of the master node (`kubeadm init`)  
- Configuration of control plane components  
- Generation of join tokens for worker nodes  

**Why it's important**:  
- Standardizes cluster creation  
- Ensures best practices for configuration  
- Reduces manual setup errors  

---

### 2. Containerd
**Definition**  
Containerd is a lightweight container runtime that manages containers on each node.  

**In Kubernetes**:  
- Acts as the runtime for pods (replacing Docker since K8s 1.24)  
- Communicates with Kubelet via **CRI**  
- Handles image pulling, container lifecycle, and storage  

**Advantages**:  
- Native CRI support  
- Stable and production-ready  

---


### 3. Calico CNI
**Definition**  
Calico is a network plugin (CNI) for Kubernetes that provides:  
- Pod-to-pod networking  
- Network policies for security  
- IP address management  

**Why it's important**:  
- Allows pods to communicate across nodes  
- Enforces network isolation rules  
- Works well with containerd  

---
### 4. Kubelet
**Definition**  
Kubelet is the agent running on every node that:  
- Communicates with the Kubernetes API Server  
- Manages pod lifecycle on the node  
- Interfaces with container runtimes via CRI  

**Key role**:  
- Ensures pods run as expected  
- Monitors node health and reports to master  

---

## 🚀 Deployment

### Prerequisites
- [Vagrant](https://www.vagrantup.com/) = 2.4.9  
- [VirtualBox](https://www.virtualbox.org/) = 7.2
- [Ansible](https://docs.ansible.com/) = 2.19 

---

### Project Structure

```bash
.
├── group_vars
│   └── all.yml                      # Global variables for all hosts
├── hosts.ini                        # Ansible inventory (masters + workers)
├── playbook.yml                     # Main playbook for the Kubernetes cluster
├── README.md                        # Project documentation
├── roles
│   ├── calico
│   │   └── tasks
│   │       └── main.yml             # Tasks to install/configure Calico
│   ├── common
│   │   └── tasks
│   │       └── main.yml             # Common tasks for all hosts
│   ├── containerd
│   │   ├── files
│   │   │   └── crictl.yaml          # crictl configuration file
│   │   ├── handlers
│   │   │   └── main.yml             # Handlers (e.g., restart service) for containerd
│   │   ├── tasks
│   │   │   └── main.yml             # Tasks to install/configure containerd
│   │   └── templates
│   │       └── config.toml.j2       # Template for containerd config.toml
│   └── kubernetes
│       └── tasks
│           └── main.yml             # Tasks to init master and join workers
└── Vagrantfile                      # VM definitions (master + workers)
```

### Setup Vagrant VMs

```bash
vagrant up
```
- Master Node: 192.168.56.10
- Worker Nodes: 192.168.56.11, 192.168.56.12
- OS: Debian Bullseye
- Resources: 2 CPUs, 2GB RAM per VM

### Run Ansible Playbook

```bash
ansible-playbook -i ansible/inventory.ini ansible/k8s-setup.yaml
```

Playbook Tasks:

- Install required packages (apt-transport-https, curl, ca-certificates, etc.)
- Add Docker and Kubernetes GPG keys & repositories
- Install containerd and configure SystemdCgroup
- Install and lock kubeadm, kubelet, kubectl
- Load kernel modules (overlay, br_netfilter) and configure sysctl
- Configure crictl
- Disable swap
- Start kubelet service
- Initialize master node with kubeadm init
- Deploy Calico CNI
- Generate and distribute join token
- Join worker nodes to cluster

### Verify Cluster

On the master node:

```bash
kubectl get nodes
```
```bash
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   61m   v1.34.1
worker1   Ready    <none>          61m   v1.34.1
worker2   Ready    <none>          61m   v1.34.1
```
```bash
kubectl get pods -A
```
```bash
kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-b45f49df6-xft7k   1/1     Running   0          4m47s
kube-system   calico-node-dv2gs                         1/1     Running   0          4m41s
kube-system   calico-node-j6hvz                         1/1     Running   0          4m48s
kube-system   calico-node-vqq89                         1/1     Running   0          4m42s
kube-system   coredns-66bc5c9577-4ht8w                  1/1     Running   0          4m47s
kube-system   coredns-66bc5c9577-jhqwq                  1/1     Running   0          4m47s
kube-system   etcd-master                               1/1     Running   0          4m53s
kube-system   kube-apiserver-master                     1/1     Running   0          4m53s
kube-system   kube-controller-manager-master            1/1     Running   0          4m53s
kube-system   kube-proxy-87w6s                          1/1     Running   0          4m42s
kube-system   kube-proxy-8nwkf                          1/1     Running   0          4m48s
kube-system   kube-proxy-blk5s                          1/1     Running   0          4m41s
kube-system   kube-scheduler-master                     1/1     Running   0          4m53s
```

### 🧪 Test du Cluster

Déployer une application test

#### Créer un déploiement nginx
```bash
cat > test-nginx.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
EOF
```
Appliquer la configuration 

```bash
kubectl apply -f test-nginx.yaml
```
Vérifier le déploiement

```bash
> kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-test   3/3     3            3           12s
```
```bash
> kubectl get pods -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE      NOMINATED NODE   READINESS GATES
nginx-test-54fc99c8d-jcqcg   1/1     Running   0          20s   10.244.235.129   worker1   <none>           <none>
nginx-test-54fc99c8d-mrqv5   1/1     Running   0          20s   10.244.189.66    worker2   <none>           <none>
nginx-test-54fc99c8d-ww57l   1/1     Running   0          20s   10.244.189.65    worker2   <none>           <none>
```
```bash
> kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        64m
nginx-service   NodePort    10.97.183.191   <none>        80:32498/TCP   29s
```
Obtenir le NodePort :

```bash
NODE_PORT=$(kubectl get svc nginx-service -o jsonpath='{.spec.ports[0].nodePort}')
echo $NODE_PORT
```

```bash
> curl http://192.168.56.10:$NODE_PORT
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 🛠️ Useful Commands

Vagrant VM Management

```bash
# Stop VMs
vagrant halt
```
```bash
# Restart VMs
vagrant reload
```
```bash
# Destroy VMs
vagrant destroy -f
```
```bash
# Check VM status
vagrant status
```

## 🧹 Nettoyage

```bash
# Destroy cluster and VMs
vagrant destroy -f
```

## Notes / Future Work


- The next tasks planned for this project include **deploying Velero** or another **backup system** for the Kubernetes cluster.
- The backup system will be **provisioned using Terraform**, allowing for infrastructure-as-code management of backup resources.