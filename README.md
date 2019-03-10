# Purpose

To provide repeatable instructions to install and configurare openshift on Centos 7 

# Description
2 node openshift cluster
- master
- node1

# Implementation (todo)
Assumes 2 VMS have been created
- master
- node1

## Configure DNS Resolution with DNSMASQ

Steps to be carried out on master and node1

```
yum install -y dnsmasq bind-utils
hostname
cp /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
vi /etc/dnsmasq.conf
address=/ocp.master.zoomer/<master node IP>
resolv-file=/etc/resolv.dnsmasq
:x
vim /etc/resolv.dnsmasq
nameserver <gateway IP>
:x
vim /etc/resolv.conf
search zoomer
nameserver 127.0.0.1
:x
systemctl stop firewalld && systemctl disable firewalld
systemctl enable dnsmasq && systemctl start dnsmasq
vim /etc/hosts
<ip master> master master.zoomer
<ip node1> node1 node1.zoomer
:x

```

Test

```
host $(hostname)
ping -c1 test.ocp.master.zoomer
ping -c1 www.google.com
```


## Configuring Docker For OpenShift Container Platform

On master
```
ssh root@192.168.11.185
ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
ll .ssh/
ssh-copy-id root@node1
ssh root@node1
exit
```

On node1
```
ssh root@192.168.11.141
ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
ll .ssh/
ssh-copy-id root@master
ssh root@master
exit
```

Setup docker on master and node1
```
yum -y install docker
cp /etc/sysconfig/docker-storage-setup /etc/sysconfig/docker-storage-setup.orig
vim /etc/sysconfig/docker-storage-setup
DEVS=vdb
VG=docker-vg
:x
lvmconf --disable-cluster
clear
docker-storage-setup
```
Confirm
```
lvs
cat /etc/sysconfig/docker-storage
```
Enable and start docker
```
systemctl enable docker
systemctl start docker
```


## Installing OpenShift with atomic-openshift-installer

On master and node1
```
yum -y install bridge-utils bind-utils git iptables-services net-tools wget
yum -y install centos-release-openshift-origin39 centos-release-paas-common
wget http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/p/python2-click-6.7-7.el7.noarch.rpm
rpm -ivh python2-click-6.7-7.el7.noarch.rpm 
yum -y install epel-release
yum --enablerepo=epel install ansible -y
yum -y install atomic-openshift-utils

```
