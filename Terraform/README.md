### Use Terraform to Create the kube-master01, kube-worker01, kube-worker02, and kube-storage01 Instances

**Install Terraform:**
```
https://www.terraform.io/downloads.html
```
  
**Create a directory on your local machine for the Terraform files:**
```
mkdir ~/kubernetes_lab
cd ~/kubernetes_lab
```

**Create the file `main.tf`:**
```
vim ~/kubernetes_lab/main.tf
```

**With the following contents:**
```
# Configure the OpenStack Provider
provider "openstack" {
  user_name   = "<YOUR_OS_USERNAME>"
  tenant_name = "<YOUR_OS_PROJECT>"
  password    = "<YOUR_OS_PASSWORD>"
  auth_url    = "<YOUR_OS_AUTH_URL>"
  region      = "<YOUR_OS_REGION>"
}
  

###############
# kube flavor #
###############
# Create the kube flavor
  
resource "openstack_compute_flavor_v2" "kube-flavor" {
  name  = "kube-flavor"
  ram   = "2048"
  vcpus = "2"
  disk  = "20"
  is_public = true
}
  
#######################
# kube-storage flavor #
#######################
# Create the kube-storage flavor
  
resource "openstack_compute_flavor_v2" "kube-storage" {
  name  = "kube-storage"
  ram   = "1024"
  vcpus = "1"
  disk  = "30"
  is_public = true
}
  
#######################
# kube security group #
#######################
# Create the kube security group
  
resource "openstack_networking_secgroup_v2" "kube_secgroup" {
  name        = "kube_secgroup"
  description = "kubernetes security group"
}
  
############################
# kube security group rule #
############################
# Add rules to the kube security group
  
# This will allow all ports inbound, this is not secure for
# a production environment
  
resource "openstack_networking_secgroup_rule_v2" "kube_secgroup_rule_1" {
  direction         = "ingress"
  ethertype         = "IPv4"
  protocol          = "icmp"
  remote_ip_prefix  = "0.0.0.0/0"
  security_group_id = "${openstack_networking_secgroup_v2.kube_secgroup.id}"
}
  
resource "openstack_networking_secgroup_rule_v2" "kube_secgroup_rule_2" {
  direction         = "ingress"
  ethertype         = "IPv4"
  protocol          = "tcp"
  port_range_min    = 1
  port_range_max    = 65535
  remote_ip_prefix  = "0.0.0.0/0"
  security_group_id = "${openstack_networking_secgroup_v2.kube_secgroup.id}"
}
  
resource "openstack_networking_secgroup_rule_v2" "kube_secgroup_rule_3" {
  direction         = "ingress"
  ethertype         = "IPv4"
  protocol          = "udp"
  port_range_min    = 1
  port_range_max    = 65535
  remote_ip_prefix  = "0.0.0.0/0"
  security_group_id = "${openstack_networking_secgroup_v2.kube_secgroup.id}"
}
  
###################################################
# kube-master01                                   #
###################################################
# Create the kube-master01 server
  
resource "openstack_compute_instance_v2" "kube-master01" {
  name            = "kube-master01"
  image_name      = "Ubuntu 20.04"
  flavor_name     = "kube-flavor"
  key_pair        = "surface" # Replace this with the name of your SSH key
  security_groups = ["kube_secgroup"]

  network {
    name = "INTERNAL_NET"
  }
}
  
# Attach a floating IP of 10.0.0.242 to the the kube-master01 server
resource "openstack_compute_floatingip_associate_v2" "kube-master01-ip" {
  floating_ip = "10.0.0.242" # Pick a valid Floating IP for your environment
  instance_id = "${openstack_compute_instance_v2.kube-master01.id}"
  fixed_ip    = "${openstack_compute_instance_v2.kube-master01.network.0.fixed_ip_v4}"
}
  
###################################################
# kube-worker01                                   #
###################################################
# Create the kube-worker01 server
  
resource "openstack_compute_instance_v2" "kube-worker01" {
  name            = "kube-worker01"
  image_name      = "Ubuntu 20.04"
  flavor_name     = "kube-flavor"
  key_pair        = "surface" # Replace this with the name of your SSH key
  security_groups = ["kube_secgroup"]

  network {
    name = "INTERNAL_NET"
  }
}
  
# Attach a floating IP of 10.0.0.239 to the the kube-worker01 server
resource "openstack_compute_floatingip_associate_v2" "kube-worker01-ip" {
  floating_ip = "10.0.0.239" # Pick a valid Floating IP for your environment
  instance_id = "${openstack_compute_instance_v2.kube-worker01.id}"
  fixed_ip    = "${openstack_compute_instance_v2.kube-worker01.network.0.fixed_ip_v4}"
}
  
###################################################
# kube-worker02                                   #
###################################################
# Create the kube-worker02 server
  
resource "openstack_compute_instance_v2" "kube-worker02" {
  name            = "kube-worker02"
  image_name      = "Ubuntu 20.04"
  flavor_name     = "kube-flavor"
  key_pair        = "surface" # Replace this with the name of your SSH key
  security_groups = ["kube_secgroup"]

  network {
    name = "INTERNAL_NET"
  }
}
  
# Attach a floating IP of 10.0.0.248 to the the kube-worker02 server
resource "openstack_compute_floatingip_associate_v2" "kube-worker02-ip" {
  floating_ip = "10.0.0.248" # Pick a valid Floating IP for your environment
  instance_id = "${openstack_compute_instance_v2.kube-worker02.id}"
  fixed_ip    = "${openstack_compute_instance_v2.kube-worker02.network.0.fixed_ip_v4}"
}
  
###################################################
# kube-storage01                                  #
###################################################
# Create the kube-storage01 server
  
resource "openstack_compute_instance_v2" "kube-storage01" {
  name            = "kube-storage01"
  image_name      = "Ubuntu 20.04"
  flavor_name     = "kube-storage"
  key_pair        = "surface" # Replace this with the name of your SSH key
  security_groups = ["kube_secgroup"]

  network {
    name = "INTERNAL_NET"
  }
}
  
# Attach a floating IP of 10.0.0.226 to the the kube-storage01 server
resource "openstack_compute_floatingip_associate_v2" "kube-storage01-ip" {
  floating_ip = "10.0.0.226" # Pick a valid Floating IP for your environment
  instance_id = "${openstack_compute_instance_v2.kube-storage01.id}"
  fixed_ip    = "${openstack_compute_instance_v2.kube-storage01.network.0.fixed_ip_v4}"
}
  
```
  
Make sure to replace the authentication info at the top along with the SSH key name and the floating IPs
Replace these items with info from your cloud
(Read the comments)
  
**Initialize the Terraform working directory:**
```
terraform init
```
  
**Note: If you get the error `Error: Failed to install providers` and it suggests:**
```
If these suggestions look correct, upgrade your configuration with the
following command:
    terraform 0.13upgrade .
```
  
**Then go ahead and run:**
```
terraform 0.13upgrade .
  
# Don't forget to rerun the init after the upgrade
terraform init
```
  
**Apply the Terraform configuration:**
```
terraform apply
```
  
[<-- Back to Main](../README.md)