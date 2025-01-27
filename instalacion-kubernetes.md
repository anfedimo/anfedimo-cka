## **Instalación de Kubernetes**

**Entorno:**

 - 2 máquinas virtuales Ubuntu 20.04
 - Kubernete versión 1.29

**Componentes:**

 - Control Plane
	 - kube-api-server
	 - ETCD
	 - Controller Manager
	 - kube-scheduler
 - Worker Node
	 - kubelet
	 - kube-proxy
	 - containerd

**Procedimiento:**

**Control Plane:**

 1. Instalar paquetería básica
```bash
sudo apt-get update  
sudo apt install apt-transport-https curl -y
```
 2. Preconfigurar networking

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

 3. Apagar Swap
```bash
sudo swapoff -a
```
 4. Instalar containerd

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

 5. Habilitar containerd como Container Runtime, replacar contenido del archivo **/etc/containerd/config.toml**

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

```

 6. Reiniciar containerd
```bash
sudo systemctl restart containerd
```
 7. Actualizar e instalar paquetería necesaria

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

 8. Descargar y agregar repositorios de kubernetes
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list    
```

 9. Instalar kubelet, kubeadm y kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
 
 11. Inicializar control plane

```bash
# Obtener ip de maquina virtual 
ip add 
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<ip de control plane>
```

 12. Habilitar cluster para cualquier usuario

```bash
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
 
 13. Instalar weave como network add-on de Kubernetes

```bash
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.29/net.yaml
```

**Worker Node:**

 1. Instalar paquetería básica
```bash
sudo apt-get update 
sudo apt install apt-transport-https curl -y
```
 1. Preconfigurar networking

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

lsmod | grep br_netfilter
lsmod | grep overlay

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

 3. Apagar swap
```bash
sudo swapoff -a
```
 4. Instalar containerd
``` bash
# Add Docker's official GPG key: 
sudo apt-get update 
sudo apt-get install ca-certificates curl 
sudo install -m 0755 -d /etc/apt/keyrings 
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc 
sudo chmod a+r /etc/apt/keyrings/docker.asc 
# Add the repository to Apt sources: 
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Update 
sudo apt-get update 
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
5. Habilitar containerd como Container Runtime, replacar contenido del archivo **/etc/containerd/config.toml**

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

```

 6. Reiniciar containerd
```bash
sudo systemctl restart containerd
```
 7. Actualizar e instalar paquetería necesaria

```bash
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
 8. Descargar y agregar repositorios de kubernetes
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list    
```

 9. Instalar kubelet, kubeadm y kubectl

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
 - Unir Worker node a Control Plane
```bash
sudo kubeadm join <ip-controlplane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:ec2ee63f8853ad42a9ec0363508d1da7b3983cdef2361efd43e20d7d85953f26
```

**Dato:** 

 - Para persistir la configuración de **swapoff -a** se debe editar el archivo **/etc/fstab**, comentar la linea asignada a swap y luego reiniciar la máquina. 
 
**Referencias:** 

• https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/

• https://docs.docker.com/engine/install/ubuntu/ 

• https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

• https://releases.ubuntu.com/20.04.6/ 

• https://cdimage.ubuntu.com/releases/20.04/release/ 
