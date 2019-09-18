# DRAFT

# Kubernetes from the bottom up

## architecture
Vagrant requirements (host machine) : RAM 8Go

## See logs on master node
```
sudo journalctl -f --unit kube-apiserver
```

## distributions
## simple app management
## chaos testing
## security
## service mesh
## Faas


Formation :

# Kubernetes self-hosted (sys-admin)
bottom up : Formation monter un cluster k8s en une semaine (ajouter les bouts les uns apres les autres)
high availability : kubeadm

# Kubernetes management (architects, developers)
- check des images docker
- kubectl

# Kubernetes for micro-services (architects, developers)
- istio
- openFaas

# Public cloud
- AWS, GCP, Azure


# Bootstrap
up the 3 VMs ```vagrant up```

ssh into master ```vagrant ssh master```

verify you can ping the 2 other using their IPs ```ping 192.168.1.11``` ```ping 192.168.1.12```

# Setup the master node
NB : done with vagrant & ansible

## Install docker
from https://docs.docker.com/install/linux/docker-ce/ubuntu/
```
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io
```

## Install kubernetes tools (kubelet, kubeadm, kubectl)

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt update && sudo apt install -y kubelet kubeadm kubectl
```

## VM networking

```
sudo ifconfig
route
sudo tcpdump -i vboxnet5 -v
ping 172.16.0.10
vagrant ssh master
sudo ifconfig
ping 172.16.0.1 (the private network interface)
```

# Bootstrap the cluster

```
sudo kubeadm init --apiserver-advertise-address=172.16.1.10 --pod-network-cidr=192.168.0.0/16
```

NB : 
- apiserver-advertise-address flag is needed because that's the IP where the machine is actually reachable
- the pod cidr is set to calico default, so we don't have to edit the chart
- maybe set the pod cidr after applying calico, just to see how errors can be fixed

## fix node networking (should be done with ansible)
```
echo "KUBELET_EXTRA_ARGS=--node-ip=172.16.2.11" | sudo tee -a /var/lib/kubelet/config.yaml
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

check :
kubelet has actually took the node-ip into account
```
ps ax | grep kubelet
```

Control plane is aware of it
```
kubectl get nodes worker-1 -o json | jq .status.addresses
```

## Install pod network (we'll use Calico)
install Calico with default settings
```
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

## demo networking
pod to pod
pod to service

kube-proxy (iptables modification)

## add oidc with keycloak (other vagrant machine)
