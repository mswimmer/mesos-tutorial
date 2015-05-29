# mesos-tutorial
Introduction of Mesos

# Prelude
Need:
* Vagrant
* VirtualBox

# Create 2 CentOS7 VMs using Vagrant
## Get the CentOS7 Vagrant box
## Create the VMs from that box
cd ~/VirtualBox VMs
vagrant init
-> creates Vagrantfile
vagrant up > vagrant_up_output

# Turning the 2 blank VMs into a Mesos Master & Mesos Slave
Check out the [official Mesos tutorial](https://docs.mesosphere.com/getting-started/datacenter/install/) and follow the "RedHat 7 / CentOS 7" instructions

## 1rst VM -> Mesos Master Node
SSH into the VM named "mesos_master"
```
$ vagrant ssh mesos_master
Welcome to your Vagrant-built virtual machine.
[root@localhost ~]#
```
### Install packages
Install Mesos
```
$ rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
$ rpm -qa | grep mesos
mesosphere-el-repo-7-1.noarch
```
Install ZooKeeper
```
$ yum -y install mesosphere-zookeeper
```
Install Marathon
```
$ yum -y install mesos marathon
```
### Configure ZooKeeper
Set the ID
/var/lib/zookeeper/ID with ID a unique integer between 1 and 255 on each node
```
$ touch /var/lib/zookeeper/1
```
ZooKeeper list of server addresses
```
$ echo "server.1=localhost:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
```
Start ZooKeeper service
```
$ systemctl start zookeeper
```
### Extra config for Mesos

ZooKeeper list of Master's IP
```
/etc/mesos/zk should remain zk://localhost:2181/mesos
```
Quorum
```
/etc/mesos-master/quorum should remain 1
```
### Disable mesos-slave service
```
$ systemctl stop mesos-slave.service
$ systemctl disable mesos-slave.service
rm '/etc/systemd/system/multi-user.target.wants/mesos-slave.service'
```
### Restart Mesos Master & Marathon services
```
$ service mesos-master restart
$ service marathon restart
```




## 2nd VM -> Mesos Slave Node
