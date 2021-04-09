# Setup an NFS Provisioner  
  
In order for us to be able to mount a volume to a Pod in our cluster with the RWX (Read Write Many) access mode, we will need to create an NFS provisioner. This will essentially create a Pod in a Deployment that will create the backend NFS shares when we create NFS backed volumes in our cluster.  
  
**On the NFS server (kube-storage01):**  
  
```
mkdir /srv/nfs/kubedata -p
apt update && apt upgrade -y && apt install -y nfs-common nfs-kernel-server
chown nobody:nogroup /srv/nfs/kubedata/
```
  
```
echo "/srv/nfs/kubedata    *(rw,sync,no_subtree_check,no_root_squash,no_all_squash,insecure)" >> /etc/exports
```
  
```
exportfs -rav
```
  
```
reboot
```
  
**On kube-master01:**  
  
**Create a `providers/nfs` directory and then create the `rbac.yaml` file:**
```
mkdir -p ~/providers/nfs
```

```
vim ~/providers/nfs/rbac.yaml
```

**Populate `rbac.yaml` with the following:**
```
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

**Create these rbac roles with:**
```
kubectl apply -f ~/providers/nfs/rbac.yaml
```

**Now create the `class.yaml` file:**
```
vim ~/providers/nfs/class.yaml
```

**And populate with the following:**
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: nfs-storage
parameters:
  pathPattern: "${.PVC.namespace}/${.PVC.annotations.nfs.io/storage-path}" # waits for nfs.io/storage-path annotation, if not specified will accept as empty string.
  onDelete: delete
```

**Create this storage class with the following:**
```
kubectl apply -f ~/providers/nfs/class.yaml
```
  
**Create the `deployment.yaml` file:**
```
vim ~/providers/nfs/deployment.yaml
```

**And populate with the following:**
***Note:*** Make sure you replace `<<NFS Server IP>>` with the actual IP of your NFS server
```
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
spec:
  selector:
    matchLabels:
      app: nfs-client-provisioner
  replicas: 1
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: rkevin/nfs-subdir-external-provisioner:fix-k8s-1.20
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-storage
            - name: NFS_SERVER
              value: <<NFS Server IP>> ## PUT IN THE IP OF YOUR NFS SERVER HERE
            - name: NFS_PATH
              value: /srv/nfs/kubedata
      volumes:
        - name: nfs-client-root
          nfs:
            server: <<NFS Server IP>> ## PUT IN THE IP OF YOUR NFS SERVER HERE
            path: /srv/nfs/kubedata
```

**Create the Deployment which will create our NFS provisioner Pod with:**
```
kubectl apply -f ~/providers/nfs/deployment.yaml
```
  
**We can verify the Deployment is up with the following:**
```
root@kube-master01:~# kubectl get deployments
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
nfs-client-provisioner   1/1     1            1           15d
```
  
**You can then verify the Pod is up with:**
```
root@kube-master01:~# kubectl get pods
NAME                                      READY   STATUS             RESTARTS   AGE
nfs-client-provisioner-68b88784f6-9pmmp   1/1     Running            4          15d
```
  
[<-- Back to Main](../README.md)