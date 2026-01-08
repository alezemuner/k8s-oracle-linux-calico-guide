# Instalação do Control Plane (Master Node)

## 1. Instalar Containerd

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io

# Configurar Cgroups (Essencial para Kubernetes)
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd

## 2. Instalar Kubeadm, Kubelet e Kubectl

# Desativar SELinux (Modo Permissive)
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config

# Adicionar Repositório Kubernetes
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.33/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# Instalar binários
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable kubelet
```

## 3. Inicializar o Cluster
Importante: O Calico requer o CIDR 192.168.0.0/16 por padrão.

```bash
# Substitua <IP-DO-SEU-SERVER> pelo IP fixo da sua VM Master
sudo kubeadm init \
  --apiserver-advertise-address=<IP-DO-SEU-SERVER> \
  --control-plane-endpoint=<IP-DO-SEU-SERVER> \
  --pod-network-cidr=192.168.0.0/16

# Configurar kubectl para o usuário
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 4. Instalar Rede (Calico CNI)
Instalar o Operador
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/tigera-operator.yaml

# Instalar os Recursos Customizados (CIDR 192.168.0.0/16)
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/custom-resources.yaml
```

Verifique se os pods do Calico estão rodando:
```bash
# Espere até todos estarem "Running"
watch kubectl get pods -n calico-system
```

## 5. Configurar Firewall (Oracle Linux)
```bash
# Portas do Kubernetes API e Etcd
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=10250-10252/tcp

# Portas do Calico (BGP e VXLAN e IPIP)
sudo firewall-cmd --permanent --add-port=179/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --permanent --add-port=5473/tcp
sudo firewall-cmd --permanent --add-protocol=ipip

# Masquerade e Interfaces
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --permanent --zone=trusted --add-interface=cali+
sudo firewall-cmd --permanent --zone=trusted --add-interface=tunl0

# Permitir tráfego da rede local (Ajuste para sua rede, ex: 192.16.1.0/24)
sudo firewall-cmd --permanent --zone=trusted --add-source=192.16.1.0/24

# Recarrega as regras
sudo firewall-cmd --reload
```
