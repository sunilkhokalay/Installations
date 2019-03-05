# Kubernetes Installation on Ubuntu

### Machine requirements
* Master Node
    * 100 GB, 4 cores, Ubuntu 64 bit OS
* Worker node
    * 200 GB, 8 cores, Ubuntu 64 bit OS

## Installation Steps
#### Add master and worker ip address in '/etc/hosts'
```
vi /etc/hosts
<master_node_ip> master
<worker_node_ip> worker
```

#### Make docker to use proxy configuration
if you are behind a proxy, follow the steps below
```
mkdir /etc/systemd/system/docker.service.d
vi /etc/systemd/system/docker.service.d/http-proxy.conf
```
Then create the content as below
```
[Service]
Environment="HTTP_PROXY=http://PROXY_IP:PORT"
Environment="HTTPS_PROXY=http://PROXY_IP:PORT"
```

#### Install Docker

```
apt-get update
apt-get install -y curl apt-transport-https
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
apt-cache policy docker-ce
sudo apt-get install -y docker-ce
sudo systemctl status docker
```

#### Install kubelet, kubeadm & kubectl

```
apt-get update
apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

#### Set no proxy

```
export no_proxy=$no_proxy,master_node_ip,worker_node_ip
```

#### Restart services

```
sudo systemctl daemon-reload
sudo systemctl enable docker kubelet
sudo systemctl restart docker kubelet
```

#### Make network bridge pesist after reboot
```
sysctl net.bridge.bridge-nf-call-iptables=1
```

#### On master node, start the cluster

```
kubeadm init --pod-network-cidr=10.244.0.0/16
```

#### Copy the join token
```
Ex:- 

kubeadm join <master_node_ip>:6443 --token 8u8x9m.54konltf44vhhs8q --discovery-token-ca-cert-hash sha256:42f5cef2d8da37c5ebad60c41c7bd461d6ff7ddb73b0c4d1edba06686c33e96d
```
#### Save kubeconfig
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### Add the following file at the end of $HOME/.bashrc
```
export KUBECONFIG=$HOME/.kube/config
```

#### Apply flannel config

```
kubectl -n kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

or 

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
```

#### On worker node, run the command generated at the time of running kubeadm init on master node.
```
Ex:- 

kubeadm join <master_node_ip>:6443 --token 8u8x9m.54konltf44vhhs8q --discovery-token-ca-cert-hash sha256:42f5cef2d8da37c5ebad60c41c7bd461d6ff7ddb73b0c4d1edba06686c33e96d
```

