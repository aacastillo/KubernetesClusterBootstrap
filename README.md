# Overview
### **Architecture**
<p align="center"> <img src="https://lucid.app/publicSegments/view/d0447fd0-4677-4418-ae84-4a05d1ca6994/image.png" alt="Architecture" /> </p>

### **Prerequisites**:
You have to have created atleast two servers. Here I spun up 3 small cloud servers using the Ubuntu 18.04 Distribution with and with 2 units of 2CPU and 2GiB each. You can spin up these servers on a cloud service like AWS or Azure and then ssh into each of them. Some training courses like [acloudguru](https://acloudguru.com/) offer cloud server sandboxes at no additional costs.

### **Installation Process**:
#### **1. Install Docker on all of the servers/nodes**
- Add the Docker Repository GPG Key

```curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -```
- Add Docker Repo

```sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"```
- Reload the apt sources list

```sudo apt-get update```
- install Docker-Community Edition

```sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu```
- Optional: Prevent auto-updates

```sudo apt-mark hold docker-ce```
<p>&nbsp;</p>

#### **2. Install the Kubernetes Componenets on all of the servers/nodes**
- Add Kubernetes Repo GPG key

```curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -```
- Add the Kubernetes Repo

```cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list deb https://apt.kubernetes.io/ kubernetes-xenial main EOF```
- Reload the apt sources list

```sudo apt-get update```
- Install the kubeadm, kubelet, and kubectl packages

```sudo apt-get install -y kubelet=1.15.7-00 kubeadm=1.15.7-00 kubectl=1.15.7-00```
- Optional: Prevent auto-updates


```sudo apt-mark hold kubelet kubeadm kubectl```
<p>&nbsp;</p>

### **Kubernetes Configuration**:
#### **1. Build a Cluster**
- Initialize the cluster on the Kube Master Server. The network CIDR will be used later for the flannel network plugin.

```sudo kubeadm init --pod-network-cidr=10.244.0.0/16```
- Set up kubeconfig for the local user on the Kube Master server. This will allow you to use kubectl when logged into the master.

```mkdir -p $HOME/.kube```
```sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config```
```sudo chown $(id -u):$(id -g) $HOME/.kube/config```
- Our first command, ```kubeadm init ``` should have outputted a kubeadm join command which would look something like the code below. Run the unique command given to you by the ```kubeadm init``` on the minion nodes to add them to your cluster.

```sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash```
- Verify that the nodes are in our cluster by running the below command in **master**, however there status should be notReady since we have not configured networking yet.

```kubectl get nodes```
<p>&nbsp;</p>

#### **2. Configure Networking**
- For networking to work we need to turn on net.bridge-nf-call-iptables on all of the nodes. This is done by setting this value to one

```echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf```
```sudo sysctl -p```
- On the master node, Kube Master, install Flannel using a YAML template

```kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml```
- Cluster should be ready, verify with

```kubectl get nodes```


