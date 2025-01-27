# Kubernetes Setup and Maintenance Documentation

Este documento detalla los pasos para la **instalación**, **actualización de IP del Control Plane** y **upgrade de Kubernetes** en un entorno de 2 máquinas virtuales (Control Plane y Worker Node) ejecutando Ubuntu 20.04.

---

## **Instalación de Kubernetes**

### **Entorno y Componentes**

- **Sistema Operativo**: Ubuntu 20.04
- **Kubernetes**: Versión 1.29

#### **Control Plane:**
- kube-api-server
- ETCD
- Controller Manager
- kube-scheduler

#### **Worker Node:**
- kubelet
- kube-proxy
- containerd

### **Pasos de Instalación**

#### **Control Plane**

1. **Instalar dependencias básicas**:
   ```bash
   sudo apt-get update  
   sudo apt install apt-transport-https curl -y
   ```

2. **Configurar networking**:
   ```bash
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   overlay
   br_netfilter
   EOF

   sudo modprobe overlay
   sudo modprobe br_netfilter

   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-iptables  = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward                 = 1
   EOF

   sudo sysctl --system
   ```

3. **Deshabilitar Swap**:
   ```bash
   sudo swapoff -a
   ```

4. **Instalar containerd**:
   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```
   Modificar `/etc/containerd/config.toml` para habilitar `SystemdCgroup`.

5. **Instalar Kubernetes**:
   ```bash
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list    
   sudo apt-get update
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

6. **Inicializar el Control Plane**:
   ```bash
   sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<ip-control-plane>
   ```

7. **Configurar acceso a Kubernetes**:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

8. **Implementar red CNI** (por ejemplo, Weave):
   ```bash
   kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml
   ```

#### **Worker Node**

1. **Configurar dependencias y networking** (similares al Control Plane).
2. **Unir Worker al Cluster**:
   ```bash
   sudo kubeadm join <ip-control-plane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

---

## **Actualizar IP del Control Plane**

1. **Detener servicios y realizar backup**:
   ```bash
   systemctl stop kubelet containerd
   sudo mv /etc/kubernetes /etc/kubernetes-bk
   sudo mv /var/lib/kubelet /var/lib/kubelet-bk
   ```

2. **Reconfigurar certificados**:
   ```bash
   sudo mkdir -p /etc/kubernetes
   sudo cp -r /etc/kubernetes-bk/pki /etc/kubernetes
   sudo rm -rf /etc/kubernetes/pki/{apiserver.*,etcd/peer.*}
   ```

3. **Reiniciar servicios y reconfigurar cluster**:
   ```bash
   systemctl start containerd
   sudo kubeadm init --control-plane-endpoint <nueva-ip> --ignore-preflight-errors=DirAvailable--var-lib-etcd
   ```

4. **Reconfigurar `kubectl`**:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

5. **Reunir los Worker Nodes**:
   ```bash
   sudo kubeadm reset
   sudo kubeadm join <nueva-ip-control-plane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
   ```

---

## **Upgrade de Kubernetes**

### **Pasos para el Control Plane**

1. **Actualizar repositorios e instalar kubeadm**:
   ```bash
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo apt update
   sudo apt install -y kubeadm=1.30.x
   ```

2. **Planificar y aplicar el upgrade**:
   ```bash
   sudo kubeadm upgrade plan
   sudo kubeadm upgrade apply v1.30.x
   ```

3. **Actualizar kubelet y kubectl**:
   ```bash
   sudo apt-get install -y kubelet=1.30.x kubectl=1.30.x
   sudo systemctl restart kubelet
   ```

### **Pasos para Worker Nodes**

1. **Actualizar kubeadm**:
   ```bash
   sudo apt-get install -y kubeadm=1.30.x
   ```

2. **Actualizar el nodo**:
   ```bash
   sudo kubeadm upgrade node
   ```

3. **Actualizar kubelet y kubectl**:
   ```bash
   sudo apt-get install -y kubelet=1.30.x kubectl=1.30.x
   sudo systemctl restart kubelet
   ```

---

## Imágenes

### Arquitectura de Kubernetes

![Arquitectura de Kubernetes](https://kubernetes.io/images/docs/components-of-kubernetes.svg)

### Flujo de instalación

![Flujo de instalación](https://d33wubrfki0l68.cloudfront.net/3451b95b6cb8ea04d5b2ca432c3061d8ab8d22cc/2a1fd/images/docs/kubeadm/kubeadm.png)
