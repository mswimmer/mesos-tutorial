# mesos-tutorial
Introduction of Mesos

# Prelude

# Create 2 CentOS7 VMs 

# Turning the 2 blank VMs into a Mesos Master & Mesos Slave
Check out the [official Mesos tutorial](https://docs.mesosphere.com/getting-started/datacenter/install/) and follow the "RedHat 7 / CentOS 7" instructions

## 1rst VM -> Mesos Master Node
### Install packages
Add yum repo
```
$ rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
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
$ echo "1" > /var/lib/zookeeper/myid
```
ZooKeeper list of server addresses
```
$ ifconfig # get IP address (interface eth0)
$ echo "server.1=MESOS_MASTER_IP:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
```
Start ZooKeeper service
```
$ systemctl start zookeeper
$ systemctl status zookeeper
zookeeper.service - Apache ZooKeeper
   Loaded: loaded (/usr/lib/systemd/system/zookeeper.service; enabled)
   Active: active (running) since Sat 2015-05-30 19:05:58 PDT; 1min 15s ago
 Main PID: 27778 (java)
   CGroup: /system.slice/zookeeper.service
           └─27778 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /opt/mesosphere/zookeeper/bin/../build/classes:/opt...

May 30 19:05:58 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com zookeeper[27778]: 2015-05-30 19:05:58,933 [myid:] - INFO  [main:Environment@...inux
May 30 19:05:58 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com zookeeper[27778]: 2015-05-30 19:05:58,933 [myid:] - INFO  [main:Environment@...md64
May 30 19:05:58 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com zookeeper[27778]: 2015-05-30 19:05:58,933 [myid:] - INFO  [main:Environment@...6_64

$ netstat -anp | grep 2181
tcp6       0      0 :::2181                 :::*                    LISTEN      27778/java
```
### Extra config for Mesos

ZooKeeper list of Master's IP
```
echo "zk://MESOS_MASTER_IP:2181/mesos" > /etc/mesos/zk
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
$ systemctl restart  mesos-master.service
$ systemctl status  mesos-master.service
mesos-master.service - Mesos Master
   Loaded: loaded (/usr/lib/systemd/system/mesos-master.service; enabled)
   Active: active (running) since Sat 2015-05-30 19:12:42 PDT; 32s ago
 Main PID: 27879 (mesos-master)
   CGroup: /system.slice/mesos-master.service
           ├─27879 /usr/sbin/mesos-master --zk=zk://10.145.6.64:2181/mesos --port=5050 --log_dir=/var/log/mesos --quorum=1 --work_dir=/var...
           ├─27889 logger -p user.info -t mesos-master[27879]
           └─27890 logger -p user.err -t mesos-master[27879]

May 30 19:12:56 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com mesos-master[27890]: I0530 19:12:56.745931 27897 master.cpp:1948] Disconnect...0467
May 30 19:12:56 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com mesos-master[27890]: I0530 19:12:56.748011 27897 master.cpp:1964] Deactivati...0467
May 30 19:12:56 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com mesos-master[27890]: I0530 19:12:56.748690 27897 master.cpp:900] Giving fram...over
$ systemctl restart marathon.service
$ systemctl status marathon.service
marathon.service - Marathon
   Loaded: loaded (/usr/lib/systemd/system/marathon.service; enabled)
   Active: active (running) since Sat 2015-05-30 19:12:56 PDT; 1min 7s ago
 Main PID: 28001 (java)
   CGroup: /system.slice/marathon.service
           ├─28001 java -Djava.library.path=/usr/local/lib:/usr/lib:/usr/lib64 -Djava.util.logging.SimpleFormatter.format=%2$s%5$s%6$s%n -...
           ├─28014 logger -p user.info -t marathon[28001]
           └─28015 logger -p user.notice -t marathon[28001]

May 30 19:13:00 d1p3920tlm-prxs-iam-a.vchslabs.vmware.com marathon[28014]: [2015-05-30 19:13:00,745] INFO Binding mesosphere.marathon....168)
```




## 2nd VM -> Mesos Slave Node
