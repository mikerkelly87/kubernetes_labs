# Kubernetes Labs

The purpose of this documentation is to get a Kubernetes cluster running in your home environment in VMs and then to go through a few examples of deploying applications.
  
**Note:** This isn't intended to be used as a production environment. This is meant to be used to learn/develop with.
  
**If you have followed my Packstack guide here:**  
 [Packstack_Labs](https://github.com/mikerkelly87/packstack_labs)
  
**Then you can follow the following guide to get your infrastructure up and running quickly using Terraform:**  
[Kubernetes Instances for Kubernetes Using Terraform](Terraform/README.md)
  
**If you don't have an Openstack cloud you can just create the following 4 VMs in your preferred hypervisor:**  
  
```
kube-master01
vCPU: 2
RAM: 2048MB
Disk: 20GB

kube-worker01
vCPU: 2
RAM: 2048MB
Disk: 20GB

kube-worker02
vCPU: 2
RAM: 2048MB
Disk: 20GB

kube-storage01
vCPU: 1
RAM: 1024
Disk: 30GB
```
  
**Then follow these documents in this order:**
  
**1) Create a Kubernetes cluster using kubeadm:**  
[Deploy the Kubernetes cluster using kubeadm](Kubernetes_Deploy/README.md)  
  
**2) Setup an NFS Provisioner to be used for backend storage to allow the use of ReadWriteMany Persistent Volumes:**  
[Deploy an NFS Provisioner](NFS_Provisioner/README.md)
  
**3) (Optional) Run through a couple of examples of deploying applications in a Kubernetes cluster:**  
[Kubernetes Labs](Kubernetes_Labs/README.md)
  
