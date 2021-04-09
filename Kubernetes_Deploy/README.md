### Use Kubeadm to deploy the Kubernetes cluster
Note: These steps were taken from the [official documentation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
  
**Login to kube-master01, kube-worker01, and kube-worker02 and run the following:**
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

sudo apt-get update && sudo apt upgrade -y && sudo apt-get install -y net-tools apt-transport-https curl nfs-common

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl docker.io
sudo apt-mark hold kubelet kubeadm kubectl docker.io

reboot
```
  
**Now that the Kubernetes packages are installed login to kube-master01 so we can deploy the control plane**
```
kubeadm init
```
This output will provide you with a `kubeadm join` command. Copy and paste this to your notes as you will need to run this command on the worker nodes to add them to the cluster.
  
**In order to interact with the cluster resources you then need to run the following on kube-master01**
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
  
**Now that kubeadm has created the control plane we need to add a pod network**
There are many different solutions you can use for pod networks listed [here](https://kubernetes.io/docs/concepts/cluster-administration/addons/) but for this lab we will use [Weave Net](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/)
  
**To deploy the Weave Net pod network run the following on your kube-master01 node**
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
  
**Now login to kube-worker01 and kube-worker02 and run the kubeadm join command you saved**
```
kubeadm join X.X.X.X:6443 --token abc123.abc123xyz1337 \
    --discovery-token-ca-cert-hash sha256:blahblahblahblahblahblah
```
  
**Once the worker nodes have joined log back into kube-master01 and run:**
```
kubectl get nodes
```
You should see output similar to:
```
root@kube-master01:~# kubectl get nodes
NAME            STATUS   ROLES                  AGE     VERSION
kube-master01   Ready    control-plane,master   32m     v1.20.1
kube-worker01   Ready    <none>                 2m13s   v1.20.1
kube-worker02   Ready    <none>                 113s    v1.20.1
```

**On kube-master01 enable kubectl autocompletion (Optional)**
```
echo 'source <(kubectl completion bash)' >>~/.bashrc
kubectl completion bash >/etc/bash_completion.d/kubectl
```

**Now let's create a test pod**
```
root@kube-master01:~# kubectl run test --image=nginx --port=80
pod/test created
```
  
**Make sure the pod is running**
```
root@kube-master01:~# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE    IP          NODE            NOMINATED NODE   READINESS GATES
test   1/1     Running   0          7m4s   X.X.X.X     kube-worker01   <none>           <none>
```
  
**Test the connection to the nginx web server in the test pod**
```
root@kube-master01:~# curl -I http://X.X.X.X
HTTP/1.1 200 OK
Server: nginx/1.19.6
Date: Fri, 08 Jan 2021 16:57:20 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 15 Dec 2020 13:59:38 GMT
Connection: keep-alive
ETag: "5fd8c14a-264"
Accept-Ranges: bytes
```
`Note: Replace 'X.X.X.X' with the actual IP of your test pod`
  
**You can delete the test pod with:**
```
root@kube-master01:~# kubectl delete pod test
pod "test" deleted
```
  
[<-- Back to Main](../README.md)