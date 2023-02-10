<!-- 
Revision: V2.0
V2.0: Updated Guide for K8 Setup
-->
***
# KUBERNETES CLUSTER INSTALLATION
***
![Sample Image](https://github.com/nikamuni/VvsKrishna25/blob/main/Screenshot%20(1).png)
## Master Node Installation(pl1vm1taack8m01)  
### Installing Docker Prerequisites packages
1. Update existing repository in the machine 
```
sudo apt-get update -y
```
2. Install prerequisite packages. These allow apt to use secure channels using HTTPS

```
sudo apt-get install ca-certificates \
curl \
gnupg \
lsb-release \
apt-transport-https \
git \
wget
```
### Install cri-dockerd
1.Clone from git repository
```
git clone https://github.com/Mirantis/cri-dockerd.git
```
2.Download installer_linux package
```
sudo wget https://storage.googleapis.com/golang/getgo/installer_linux
```
3.Update execute permissions to installer_linux package
```
sudo chmod +x installer_linux
```
4.Run installer_linux package
```
sudo ./installer_linux
```
5.Reload amd user bash profile
```
source /home/amd/.bash_profile
```
6.Change directory to cri-dockerd
```
cd cri-dockerd/
```
7.Create bin directory
```
mkdir bin
```
8.Build cri-dockerd binary file
```
go build -o bin/cri-dockerd
```
9.Install cri-dockerd package
```
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
```
10.Copy cri-dockerd files on to /etc/system/system/ 
```
sudo cp -a packaging/systemd/* /etc/systemd/system
```
11.Update cri-docker service
```
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```
12.Reload the cri-docker service
```
sudo systemctl daemon-reload
```
13.Enable cri-docker service
```
sudo systemctl enable cri-docker.service
```
### Install Docker Engine

1.Create /etc/apt/keyrings directory
```
sudo mkdir -p /etc/apt/keyrings
```
2.Add the GPG key from the official Docker repository
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
3.Add the Docker repository and update package information to apt sources list
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
4.Download docker-ce package
```
sudo apt-get install docker-ce
```
5.Enable cri-docker socket
```
sudo systemctl enable --now cri-docker.socket
```
## Kubernetes Installation

1.Disable SWAP permanently
```
sudo swapoff -aff –a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
2.Add the GPG key from the official Kubernetes repository
```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
3.Add the Docker repository and update package information to apt sources list
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \ sudo tee /etc/apt/sources.list.d/kubernetes.list
```
4.Update existing repository in the machine 
```
sudo apt-get update -y
```
5.Download kuberenets components
```
sudo apt-get install -y kubelet kubeadm kubectl
```
6.Update following sysctl params required by setup
```
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
``` 
## Master Node Initialization (pl1vm1taack8m01)  
### Run this section on Kubernetes master node
## Initialize cluster using kubeadm 
### Initialize using  [ kubeadm init –options ]
```
sudo kubeadm init  --pod-network-cidr=192.168.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
Pod-network-cdir
cri-socket
```
## Configure cluster
### Configure ‘config’ to use cluster by user
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Deploy Container Networking
Deploy POD network
```
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O \
kubectl apply -f calico.yaml
```
## Verify Cluster status
1.Verify master node status
```
kubectl get nodes
```
Note: Master Node Should be *Ready* State

2.Verify Pods 
```
kubectl get pods –A
```
Note: All PODS should be *Running* state

## Worker Nodes Installation (pl1vm1taack8w01,w02)  
### Installing Docker Prerequisites packages
1.Update existing repository in the machine 
```
sudo apt-get update -y
```
2.Install prerequisite packages. These allow apt to use secure channels using HTTPS
```
sudo apt-get install ca-certificates \
curl \
gnupg \
lsb-release \
apt-transport-https \
git \
wget
```
## Install cri-dockerd
1.Clone from git repository
```
git clone https://github.com/Mirantis/cri-dockerd.git
```
2.Download installer_linux package
```
sudo wget https://storage.googleapis.com/golang/getgo/installer_linux
```
3.Update execute permissions to installer_linux package
```
sudo chmod +x installer_linux
```
4.Run installer_linux package
```
sudo ./installer_linux
```
5.Reload amd user bash profile
```
source /home/amd/.bash_profile
```
6.Change directory to cri-dockerd
```
cd cri-dockerd/
```
7.Create bin directory
```
mkdir bin
```
8.Build cri-dockerd binary file
```
go build -o bin/cri-dockerd
```
9.Install cri-dockerd package
```
sudo install -o root -g root -m 0755 bin/cri-dockerd /usr/local/bin/cri-dockerd
```
10.Copy cri-dockerd files on to /etc/system/system/ 
```
sudo cp -a packaging/systemd/* /etc/systemd/system
```
11.Update cri-docker service
```
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
```
12.Reload the cri-docker service
```
sudo systemctl daemon-reload
```
13.Enable cri-docker service
```
sudo systemctl enable cri-docker.service
```
### Install Docker Engine
1.Create /etc/apt/keyrings directory
```
sudo mkdir -p /etc/apt/keyrings
```
2.Add the GPG key from the official Docker repository
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
3.Add the Docker repository and update package information to apt sources list
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
4.Download docker-ce package
```
sudo apt-get install docker-ce
```
5.Enable cri-docker socket
```
sudo systemctl enable --now cri-docker.socket
```
### Kubernetes Installation
1.Disable SWAP permanently
```
sudo swapoff -aff –a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
2.Add the GPG key from the official Kubernetes repository
```
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
3.Add the Docker repository and update package information to apt sources list
```
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \ sudo tee /etc/apt/sources.list.d/kubernetes.list
```
4.Update existing repository in the machine 
```
sudo apt-get update -y
```
5.Download kuberenets components
```
sudo apt-get install -y kubelet kubeadm kubectl
```
6.Update following sysctl params required by setup
```
cat << EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 
```
## Worker Nodes Installation (pl1vm1taack8w03)  
### Docker Engine Installation
#### Using cri-dockerd to integrate docker Engine with Kubernetes. 
1.Install Docker
```
sudo zypper install docker
```
2.Enable docker service
```
sudo systemctl enable docker.service
```
3.Start docker service
```
sudo systemctl start docker.service
```
4.Check docker service status
```
sudo systemctl status docker.service
```

## Kubernetes Installation
### Using kubeadm tool for Kubernetes installation.

1.Disable SWAP permanently:
```
sudo swapoff -aff –a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
2.Update following sysctl params required by setup
```
sudo sysctl net.ipv4.ip_forward
sudo sysctl net.ipv4.conf.all.forwarding
sudo sysctl net.bridge.bridge-nf-call-iptables
```
3.Add Kubernetes repo on to the machine
```
sudo zypper addrepo --type yum --gpgcheck-strict –refresh 
```
*https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64 google-k8s

4.Import GPG key
```
sudo rpm --import https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
sudo rpm --import https://packages.cloud.google.com/yum/doc/yum-key.gpg 
```

5.Reload zypper packages
```
sudo zypper refresh google-k8s
```

6.Install kubernetes packages
```
sudo zypper in kubelet-1.26.0-0 kubernetes-cni kubeadm-1.26.0-0 cri-tools kubectl-1.26.0-0 socat
```

## Kubernetes Cluster setup
### Join worker nodes to the cluster
1.Generate token on the Master node 
```
sudo kubeadm token create --print-join-command
```
*Note: The output of above command should be executed on all worker nodes to join them into the cluster.

2.kubeadm token command execution output looks like below
``` 
sudo kubeadm join 10.194.32.11:6443 --token 3o2vpa.3sx2dju6hsipfmjp \
--discovery-token-ca-cert hash sha256:ee1cbbe12b2d54bb1c64cf57d03da3c6f254109984bcaa4fa925a834a6f216bb \
	--cri-socket=unix:///var/run/cri-dockerd.sock
```
3.Run Kubernetes get nodes command on Master node to see the status of worker nodes. Must wait until all worker nodes in Ready State to use the cluster. 

## Kubernetes Cluster Administration
### Kubeconfig file Generation 
Generate a certs, public and private RSA key pair
1.Create certs directory and change to that directory
```
mkdir certs
cd certs/
```
2.Generate openssl certs
```
openssl genrsa -out plexus.key 2048
```
```
openssl req -new -key plexus.key -out plexus.csr -subj "/CN=plexus/O=plexus"
```
```
sudo openssl x509 -req -in plexus.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out plexus.crt -days 30
```
3.Copy kubeconfig file to the user config
``` 
cp /etc/kubernetes/admin.conf plexus.conf
sudo cp /etc/kubernetes/admin.conf plexus.conf
kubectl get pods --kubeconfig=/home/amd/kube_deploy/certs/plexus.conf
```

## Role Based Access Control
### Normal Role for user.
1.Apply role for user
```
kubectl create -f rbac_plexus.yaml
```
## Cluster role for user.
1.Apply cluster role for user
```
kubectl create -f cluster_rbac_plexus.yaml
```
*Note: Sample YAML has been added in the backup slides.

2.Run kubectl api-resources | grep role command on Master node to see the status of roles.

## Labels and Topologies for the nodes
### Applying Labels
Workload will be allocated in either GPU or CPU worker nodes depending on whether they require GPU architecture or not.
1.GPU worker nodes must be labeled with node-role.kubernetes.io/plexus-worker-type=plexus-gpu-worker.
2.CPU worker nodes must be labeled with node-role.kubernetes.io/plexus-worker-type=plexus-cpu-worker.

### Apply label for cpu worker nodes:
```
kubectl label nodes pl1vm1taack8w01 node-role.kubernetes.io/plexus-worker-type=plexus-cpu-worker
kubectl label nodes pl1vm1taack8w02 node-role.kubernetes.io/plexus-worker-type=plexus-cpu-worker
```
## Topology
*To provide pod topology affinity, every node in the cluster must be labeled with topology. Kubernetes.io/zone=X, where X is the node zone identifier.
### Apply topology label for worker nodes:
```
kubectl label nodes pl1vm1taack8w01 topology.kubernetes.io/zone=aac
kubectl label nodes pl1vm1taack8w02 topology.kubernetes.io/zone=aac
```
*Run kubectl get nodes --show-labels command on Master node to List the nodes along with the labels in the cluster.

## MetalLB Configuration
### Installing the MetalLB
1.Download MetalLB
```
curl -s https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```
2.Deploying MetalLB
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
```
3.Configuration of MetalLB
```
https://metallb.universe.tf/configuration/#layer-2-configuration
```
4.Verification of MetalLB
*Expose the deployment pods to external network:
```
kubectl expose deploy nginx --port 80 --type LoadBalancer
kubectl expose deploy nginx-deployment --port 80 --type LoadBalancer
```
b.Run kubectl get svc to check if External-IP is assigned.
