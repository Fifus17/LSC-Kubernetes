# LSC-Kubernetes

Command to run nfs server:
```bash
helm install nfs-server kvaps/nfs-server-provisioner \
  --set persistence.enabled=true \
  --set persistence.size=1Gi \
  --set storageClass.name=nfs-sc \
  --set storageClass.defaultClass=false

```

Then from the `deployment` directory run:

```bash
kubectl apply -R -f .
```

Last step is running:

```bash
minikube service nginx-service
```

Then we can go to `http://127.0.0.1:50731` ans we will see:
![alt text](image.png) in our browser.

##  Component Roles Summary
```
+----------------------------+
|      Web Browser          |
|  (Access via NodePort)    |
+------------+--------------+
             |
             v
+------------+--------------+
|     NodePort Service      |
|    (Exposes Nginx Pod)    |
+------------+--------------+
             |
             v
+------------+--------------+
|     Nginx Deployment      |
|  (Serves content from PVC)|
+------------+--------------+
             |
             v
+------------+--------------+
|    PersistentVolumeClaim  |
|          (nfs-pvc)        |
+------+-----------+--------+
       |           ^
       v           |
+------+--+    +---+----------+
| Init Job|    | NFS Server   |
| Writes  |    | + Provisioner|
| index.html    | (via Helm)  |
+--------------+-------------+
```

| Component                     | Role                                                                 |
|------------------------------|----------------------------------------------------------------------|
| **Minikube**                 | Local Kubernetes cluster used to host and run all resources          |
| **Helm Chart: NFS Server**   | Deploys an NFS server and dynamic provisioner for RWX storage        |
| **StorageClass (`nfs-sc`)**  | Custom StorageClass that provisions NFS-backed PersistentVolumes     |
| **PersistentVolumeClaim**    | Dynamically bound to an NFS volume; used to share data between Pods  |
| **Init Job (`init-web-content`)** | A one-time Job that writes `index.html` to the shared volume       |
| **Nginx Deployment**         | A Pod that serves static web content from the shared NFS volume      |
| **NodePort Service**         | Exposes Nginx to external access via Minikube on a specific port     |
