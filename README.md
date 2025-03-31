# kubernetes_one_node
install kubernetes one node on ubunry server


Ubuntu 22.04 server

Change ip to static 

vi /etc/cloud/cloud.cfg.d/90-installer-network.cfg
Ten wpis ip co ostatni

network: {config: disabled}

vi /etc/netplan/50-cloud-init.yaml


network:
  version: 2
  ethernets:
    ens160:
      addresses:
        - 192.168.1.10/24
      dhcp6: false
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.1
          - 8.8.8.8
        search: []
      optional: true


sudo netplan try
sudo netplan apply

sudo su -

sudo apt update
Reboot

sudo hostnamectl set-hostname "k8s-master.nvtienanh.local"

vi etc hosts
127.0.0.1 localhost
127.0.1.1 kubernetes
192.168.1.10 k8s-master.nvtienanh.local k8s-master
sudo swapoff -a

haszować w FSTAB swap
vi /etc/fstab

#/swap.img      none    swap    sw      0       0

sudo mount -a
free -h

 /etc/selinux/config


sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF


sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system


sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.asc] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo rm -rf /etc/apt/keyrings
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.asc >/dev/null

vi sudo modprobe br_netfilter
dodać 
net.bridge.bridge-nf-call-iptables = 1
sudo sysctl -p



sudo apt update

sudo apt install -y kubelet kubeadm kubectl

sudo apt-mark hold kubelet kubeadm kubectl



I jak bedzie ok to

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join k8s-master.nvtienanh.local:6443 --token x9e3s5.hc5816zur12ho0vy \
        --discovery-token-ca-cert-hash sha256:26c9782d0e6a0978ecaccb047147ff9df25b74d36b123b53fbef1119d2f750d9 \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master.nvtienanh.local:6443 --token x9e3s5.hc5816zur12ho0vy \
        --discovery-token-ca-cert-hash sha256:26c9782d0e6a0978ecaccb047147ff9df25b74d36b123b53fbef1119d2f750d9


mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl cluster-info
kubectl get nodes

curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O

kubectl apply -f calico.yaml

kubectl get pods -n kube-system

NAME         STATUS   ROLES           AGE     VERSION
kubernetes   Ready    control-plane   8m11s   v1.29.15


Teraz jak chcemy miec jeden node master I worker to robimy:

kubectl label nodes kubernetes node-role.kubernetes.io/worker=""
kubectl get nodes![image](https://github.com/user-attachments/assets/28b12e94-5de9-4356-ba37-337cf52ad6b9)
