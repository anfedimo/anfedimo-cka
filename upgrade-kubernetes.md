## **Upgrade de Kubernetes**

**Entorno:**

 - 2 máquinas virtuales Ubuntu 20.04
 - Kubernete versión 1.29 upgrade a versión 1.30
  
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
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg 
sudo apt update
```
 1. Verificamos las versiones de kubernetes disponible

```bash
sudo apt-cache madison kubeadm
```

 3. Actualizamos kubeadm
```bash
sudo apt-mark unhold kubeadm && \ 
sudo apt-get update && sudo apt-get install -y kubeadm='1.30.2-1.1’ && \ 
sudo apt-mark hold kubeadm
```
 4. Verificamos el proceso de upgrade

```bash
sudo kubeadm upgrade plan
```

 5. Actualizamos nuestra versión de cluster

```
sudo kubeadm upgrade apply v1.30.x
```

 6. . Ponemos el Control Plane en modo mantenimiento para poder actualizar el kubelet y kubectl
```bash
kubectl drain <control-plane> --ignore-daemonsets
```
 7. Actualizamos el kubelet y kubectl

```bash
# replace x in 1.30.x-* with the latest patch version 
sudo apt-mark unhold kubelet kubectl && \ 
sudo apt-get update && sudo apt-get install -y kubelet='1.30.x-*' kubectl='1.30.x-*' && \ 
sudo apt-mark hold kubelet kubectl
```

 8. Reiniciamos el kubelet
```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet    
```

 9. Habilitamos nuevamente el control plane

```bash
kubectl uncordon <control-plane>
```


**Worker Node:**

 1. Descargar y actualizar repositorio de kubernetes
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list 
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg 
sudo apt update
```
 2. Verificamos las versiones de kubernetes disponible

```bash
sudo apt-cache madison kubeadm
```

 3. Actualizamos la versión de kubelet
```bash
sudo kubeadm upgrade node
```
 4. Colocamos en mantenimiento el worker node
``` bash
kubectl drain <worker-node> --ignore-daemonsets
```
5. Actualizamos el kubelet y el kubectl

```
# replace x in 1.30.x-* with the latest patch version 
sudo apt-mark unhold kubelet kubectl && \ 
sudo apt-get update && sudo apt-get install -y kubelet='1.30.x-*' kubectl='1.30.x-*' && \ 
sudo apt-mark hold kubelet kubectl

```

 6. Reiniciamos el kubelet
```bash
sudo systemctl daemon-reload 
sudo systemctl restart kubelet
```

 8. Habilitamos nuevamente el worker node
```bash
kubectl uncordon <worker-node>   
```


**Referencias:** 

• https://kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/ 
• https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/ 
• https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linuxnodes/

