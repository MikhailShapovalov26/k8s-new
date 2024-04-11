https://github.com/aynels/ansible-k8s-install-playbook/blob/main/k8s-install-playbook.yaml

```
sudo apt-get update -y
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update -y
sudo apt-get install -y kubelet kubeadm kubectl containerd
sudo apt-mark hold kubelet kubeadm kubectl

sudo -i
modprobe br_netfilter
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf

# get master ip for --apiserver-advertise-address
ip a

sudo kubeadm init \
 --apiserver-advertise-address=10.128.0.8 \
 --pod-network-cidr 10.244.0.0/16 \
 --apiserver-cert-extra-sans=158.160.119.24
 --control-plane-endpoint=cluster_ip_address

10.128.0.26 — этот адрес слушает apiserver, необходимо взять внутренний айпишник вм, которая будет мастером (мастер ноды)
10.244.0.0/16 — сеть для подов
51.250.94.68 — внешний адрес, куда будем подключаться с помощью kubectl
cluster_ip_address — кластерный IP-адрес (адрес LoadBalancer) для HA control plane, если 1 мастер, то убираем


  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config


worker nodes
sudo kubeadm join 10.128.0.28:6443 --token zvxm7y.z61zq4rzaq3rtipk \
        --discovery-token-ca-cert-hash sha256:9b650e50a7a5b6261746684d033a7d6483ea5b84db8932cb70563b35f91080f7


kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Установка через kubespray

```bash
sudo apt-get update -y
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update -y
sudo apt-get install git pip python3.11 -y

sudo -i
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.11 get-pip.py

# RETURN TO USER
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/
pip3.11 install -r requirements.txt

# Copy ``inventory/sample`` as ``inventory/mycluster``
cp -rfp inventory/sample inventory/mycluster

# Update Ansible inventory file with inventory builder
declare -a IPS=(10.128.0.5 10.128.0.3 10.128.0.13)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3.11 contrib/inventory_builder/inventory.py ${IPS[@]}

# Copy private ssh key to ansible host
scp -i ~/.ssh/yandex yandex yc-user@51.250.76.222:.ssh/id_rsa
sudo chmod 600 ~/.ssh/id_rsa

ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b -v &
```

https://medium.com/@murat.bilal/setting-up-kubernetes-cluster-with-multiple-control-plane-nodes-behind-haproxy-9384c6c040d8


Дополнительно, если необходимо развернуть несколько Мастер нод, то необходимо в inventroy добавить
```
[k8s_masters]
k8s-master1
k8s-master2
k8s-master3
```
через terraform не реализовывал данный функционал.

Дополнительно для будущих

https://github.com/BigKAA/youtube/blob/master/kubeadm/ha_cluster.md

https://habr.com/ru/companies/flant/articles/427283/

https://github.com/BigKAA/youtube/blob/master/kubeadm/first_control_node.md