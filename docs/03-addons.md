# Pós-Instalação: Storage e LoadBalancer

Configurações para habilitar volumes persistentes e IPs externos em ambiente virtual.

## 1. Storage Class (Local Path Provisioner)

Permite criar PVCs dinâmicos usando o disco local do nó.

```bash
# Instalar
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml

# Verificar
kubectl get sc
```

## 2. MetalLB (LoadBalancer)
Permite que serviços do tipo LoadBalancer recebam um IP da sua rede local.

```bash
# Habilitar Strict ARP
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system

# Instalar Manifestos
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml

Configuração de Rede (IPAddressPool)
Crie um arquivo metallb-config.yaml. Defina um range de IPs da sua rede que não esteja no DHCP.

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: meu-pool-ips
  namespace: metallb-system
spec:
  addresses:
  - 192.16.1.245-192.16.1.250 # <--- AJUSTE PARA SUA REDE

apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: meu-anuncio-l2
  namespace: metallb-system
spec:
  ipAddressPools:
  - meu-pool-ips
``` 

Aplique:
```bash
kubectl apply -f metallb-config.yaml
```

