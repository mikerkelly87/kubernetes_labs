# Kubernetes Labs  
  
These labs aren't necessarily deploying applications the way you might in a production environment. The purpose of these are just have a couple of examples to refer back to.

**LAB 1: Single Pod Deployment**  
In this lab we will deploy a LEMP stack in single Pod Deployemnts on a single node  
[LAB 1 -->](Labs/single.md)  
  
**LAB 2: Redundant Pod Application**  
In this lab we will deploy a Wordpress solution with redundant Deployments which will run in multiple Pods across multiple nodes  
[LAB 2 -->](Labs/redundant.md)  
  
---
**At any time in these labs you can get shell access to the container in a pod using the following process:**
```
root@kube-master01:~# kubectl get pods
NAME                                     READY   STATUS      RESTARTS   AGE
mysql-67949d47f5-4cjmh                   1/1     Running     0          73s

root@kube-master01:~# kubectl exec -it mysql-67949d47f5-4cjmh -- /bin/sh
# hostname
mysql-67949d47f5-4cjmh
#
```
  
**If you have multiple containers in a single pod you will probably see the following message:**
```
root@kube-master01:~# kubectl get pods
NAME                                     READY   STATUS      RESTARTS   AGE
webapp-wordpress-fb79dc968-c7v4s         2/2     Running     0          3m2s

root@kube-master01:~# kubectl exec -it webapp-wordpress-fb79dc968-c7v4s -- /bin/sh
Defaulting container name to nginx.
Use 'kubectl describe pod/webapp-wordpress-fb79dc968-c7v4s -n default' to see all of the containers in this pod.
#
```

**In this example there are 2 containers in this pod, nginx, and php-fpm:**
```
root@kube-master01:~# kubectl describe pod webapp-wordpress-fb79dc968-c7v4s
Name:         webapp-wordpress-fb79dc968-c7v4s
...
Containers:
  nginx:
    Container ID:   docker://d104d1dde62fcddd1afd46993de0d379a3539a3772811dc942c5787b6eaecc45
    Image:          nginx
...
  php-fpm:
    Container ID:  docker://52c0a84cf8a552754875a2bf95f683148729ea0ba3c7c18caf9f78743b942494
    Image:         mikerkelly87/php-fpm-ubuntu:latest
...
```
  
**In this case you should specify which container you want to get shell access to:**
```
root@kube-master01:~# kubectl get pods
NAME                                     READY   STATUS      RESTARTS   AGE
webapp-wordpress-fb79dc968-c7v4s         2/2     Running     0          3m2s

root@kube-master01:~# kubectl exec -it webapp-wordpress-fb79dc968-c7v4s --container nginx -- /bin/sh
#

OR

root@kube-master01:~# kubectl exec -it webapp-wordpress-fb79dc968-c7v4s --container php-fpm -- /bin/sh
#
```
    
[<-- Back to Main](../README.md)