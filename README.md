# Instalação de Kubernetes em Oracle Linux com Calico

Este repositório documenta a instalação passo a passo de um cluster Kubernetes (v1.30+) utilizando **Oracle Linux 9** e **Calico CNI**.
O guia cobre desde a preparação do Sistema Operacional até a configuração de Storage Dinâmico e LoadBalancer para ambientes on-premise (VMs).

## 📋 Arquitetura do Lab

- **OS:** Oracle Linux 9 (Unbreakable Enterprise Kernel)
- **Container Runtime:** Containerd
- **CNI (Rede):** Calico (Network CIDR: `192.168.0.0/16`)
- **Storage:** Rancher Local Path Provisioner
- **LoadBalancer:** MetalLB (Layer 2)

## 📚 Documentação

Siga os guias na ordem abaixo:

1. [Instalação do Control Plane (Master)](docs/01-control-plane.md)
2. [Configuração dos Worker Nodes](docs/02-worker-nodes.md)
3. [Add-ons: Storage Class & MetalLB](docs/03-addons.md)

## 🛠️ Pré-requisitos Gerais (Todos os Nós)

Todos os nós (Master e Workers) devem ter estas configurações aplicadas antes da instalação:

### Kernel e Rede
```bash
# Carregar módulos
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

```bash
# Parâmetros Sysctl
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

```bash
#Desativar Swap
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```
