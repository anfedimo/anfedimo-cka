## **Update IP de Control Plane Kubernetes**

**Procedimiento:**

**Control Plane:**

 1. Detener kubelet y containerd 
```bash
systemctl stop kubelet containerd
```
 2. Obtener backup de datos de kubernetes y kubelet

```bash
sudo mv /etc/kubernetes /etc/kubernetes-bk
sudo mv /var/lib/kubelet /var/lib/kubelet-bk
```

 3. Respaldar los certificados
```bash
sudo mkdir -p /etc/kubernetes
sudo cp -r /etc/kubernetes-bk/pki /etc/kubernetes 
sudo rm -rf /etc/kubernetes/pki/{apiserver.*,etcd/peer.*}
```
 4. Reiniciar containerd

```bash
systemctl start containerd
```

 5. Re-inicializamos el cluster, **si les da error es porque algunos puertos aún se mantienen en listen solamente le dan Kill a los procesos asociados**

```
sudo kubeadm init --control-plane-endpoint <nueva-ip> --ignore-preflight-errors=DirAvailable--var-lib-etcd
```

 6. Sobreescribimos nuestro kubeconfig
```bash
mkdir -p $HOME/.kube 
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


**Worker Node:**

 1. Reiniciamos configuraciones
```bash
sudo kubeadm reset
```
2.   Unimos nuevamente el Worker node al Control Plane
```bash
sudo kubeadm join <ip-controlplane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:ec2ee63f8853ad42a9ec0363508d1da7b3983cdef2361efd43e20d7d85953f26
```

