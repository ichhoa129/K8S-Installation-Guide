# Kubernetes installation scripts

## Prerequisites
1. Run ``` sudo su ``` on all vm

## Setup loadbalancer (K8S-LB)
```sh
apt update && apt install -y haproxy
vi /etc/haproxy/haproxy.cfg
# append the lines in file haproxy.cfg to the bottom of file /etc/haproxy/haproxy.cfg on server
systemctl restart haproxy
systemctl enable haproxy

#optional
systemctl status haproxy

```
## Install containerd (All nodes)
```sh
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sudo sysctl --system
sudo apt install containerd -y
sudo mkdir /etc/containerd
sudo containerd config default > /etc/containerd/config.toml
sudo swapoff -a
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

## Install kubeadm, kubelet, kubectl (All nodes)
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# optional
sudo systemctl enable --now kubelet
```

## Init cluster (Master Node 1)
```sh
kubeadm init --control-plane-endpoint="<LB_IP>:6443" --upload-certs --apiserver-advertise-address=<CURRENT_MASTER_IP> --pod-network-cidr=192.168.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml

```

## Join Masters (Master node 2,3)
```sh
kubeadm join <LB_IP>:6443 --token 9a2d7l.ksfy7ae29073ulpa \
	--discovery-token-ca-cert-hash sha256:16c290a8983b7051491ecace1227e83d825ad752bf69485cca310228394b4aba \
	--control-plane --certificate-key 1d25b484aeb7c672708ee90bd365df0e3563dc76dfb22e3390f3c548846d18ad

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join Cluster (Worker nodes)
```sh
kubeadm join <LB_IP>:6443 --token 9a2d7l.ksfy7ae29073ulpa \
	--discovery-token-ca-cert-hash sha256:16c290a8983b7051491ecace1227e83d825ad752bf69485cca310228394b4aba

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Add master node
```sh
kubeadm join lb_ip:6443 --token oylvmu.12pwimke0blaaeji --discovery-token-ca-cert-hash sha256:303a791ef0bdaeb3a3b54ca80f8f4831dff6d0bb1c43c664d9102c9ec569ef61 --control-plane --certificate-key 3b4da12cd25d1c1e7a47abcb908c73405c4abd5e542f99692d8f1b9d368d307a
```

## Add worker node
```sh
kubeadm join lb_ip:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

# Other commands
## Get token list for joining in worker nodes
```sh
kubeadm token list
> 9a2d7l.ksfy7ae29073ulpa
```

## Create token
```sh
kubeadm token create
```

## Discovery token ca cert hash
```sh
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
openssl dgst -sha256 -hex | sed 's/^.* //'

> 16c290a8983b7051491ecace1227e83d825ad752bf69485cca310228394b4aba
```
