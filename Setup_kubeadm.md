# Set up kubernetes with kubeadm

## Init centos 7 server
```bash
# Update software
yum update -y

# Install docker
yum install -y yum-utils device-mapper-persistent-data lvm2

yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

yum install -y docker
systemctl enable docker
systemctl start docker

# instal kubernetes
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Install ufw
yum install epel-release -y
yum install -y ufw
echo y | ufw enable

for index in 4 3 2 1; do
  echo y | ufw delete $index
done

ufw allow from 0.0.0.0/0

echo y | ufw enable

# Instal kubelet, kubeadm and kubectl
sudo yum install -y kubelet kubeadm kubectl
systemctl enable kubelet
systemctl start kubelet

# Security
setenforce 0
swapoff -a
echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables
systemctl stop firewalld
systemctl disable firewalld
```

## Init kubernetes
```bash
kubeadm init --kubernetes-version=v1.20.4 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.1.40
---
kubeadm join 192.168.1.40:6443 --token xxx \
    --discovery-token-ca-cert-hash sha256:yyy
```

## Config bash
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## Test
```bash
kubectl get nodes
---
NAME      STATUS     ROLES                  AGE     VERSION
master    NotReady   control-plane,master   5m43s   v1.20.4
server1   Ready      <none>                 4m28s   v1.20.4
server2   Ready      <none>                 62s     v1.20.4
```

## Install networking
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```