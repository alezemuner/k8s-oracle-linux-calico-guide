Siga estes passos em cada máquina que será um Worker.

## 1. Instalação Base
Execute os mesmos passos do Master para:
1. Pré-requisitos de Kernel e Swap (Ver README).
2. Instalação e configuração do `containerd` (Ver 1.Instalar Containerd no 01-control-plane.md).
3. Instalação dos binários `kubeadm`, `kubelet` e `kubectl` (Ver 1.Instalar Containerd no 01-control-plane.md).

## 2. Configurar Firewall do Worker
O Worker tem portas diferentes. Ele não precisa expor a API (6443), mas precisa expor portas para serviços (NodePort).

```bash
# Porta do Kubelet e Calico
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=179/tcp
sudo firewall-cmd --permanent --add-port=4789/udp
sudo firewall-cmd --permanent --add-port=5473/tcp
sudo firewall-cmd --permanent --add-protocol=ipip

# Portas para NodePort Services (Acesso externo as aplicações)
sudo firewall-cmd --permanent --add-port=30000-32767/tcp

# Masquerade e Rede
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --permanent --zone=trusted --add-interface=cali+
sudo firewall-cmd --permanent --zone=trusted --add-interface=tunl0
# Permitir tráfego da rede local (Ajuste para sua rede, ex: 192.16.1.0/24)
sudo firewall-cmd --permanent --zone=trusted --add-source=192.16.1.0/24
sudo firewall-cmd --reload
```

## 3. Ingressar no Cluster (Join)
No Master Node, gere o comando de token:
```bash
kubeadm token create --print-join-command
```
Copie a saída (começa com kubeadm join...) e execute no Worker Node como sudo.

## 4. Verificação
Volte ao Master e rode:
```bash
kubectl get nodes
```
Aguarde alguns minutos até que o status mude de NotReady para Ready (isso acontece assim que o Calico sobe no worker).
