# mesos-tutorial
Introduction of Mesos

# Prelude


# Create 2 CentOS7 VMs 

## Cloning VM from CentOS 7 Vagrant template
I used:
* Mesos Master node: 10.145.6.64 / d1p3920-charles-mesos-master.vchslabs.vmware.com
* Mesos Slave  node: 10.145.6.68 / d1p3920-charles-mesos-slave.vchslabs.vmware.com

## Set up network configuration
Check out [this gist](https://gist.github.com/craimbert/1fb6c4dd296c84f3a253)

# Turning the 2 blank VMs into a Mesos Master & Mesos Slave
Check out the [official Mesos tutorial](https://docs.mesosphere.com/getting-started/datacenter/install/) and follow the "RedHat 7 / CentOS 7" instructions

# Install 1rst VM -> Mesos Master Node
## Install packages
Add yum repo
```
$ rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
```
Install ZooKeeper
```
$ yum -y install mesosphere-zookeeper
```
Install Mesos & Marathon
```
$ yum -y install mesos marathon
```
## ZooKeeper
### Configure ZooKeeper
Set the ID
/var/lib/zookeeper/myid with an unique integer between 1 and 255 on each node
```
$ echo "1" > /var/lib/zookeeper/myid
```
ZooKeeper list of server addresses
```
$ ifconfig eth0 # get IP address (interface eth0)
$ echo "server.1=MESOS_MASTER_IP:2888:3888" >> /etc/zookeeper/conf/zoo.cfg
```
### Start ZooKeeper service
```
$ systemctl start zookeeper
```
### Test ZooKeeper service
```
$ ps -aux | grep zookeeper
root  1138  java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /opt/mesosphere/zookeeper/bin/...
```
```
$ systemctl status zookeeper
zookeeper.service - Apache ZooKeeper
   Loaded: loaded (/usr/lib/systemd/system/zookeeper.service; enabled)
   Active: active (running) since Sun 2015-05-31 18:26:57 PDT; 18min ago
 Main PID: 1138 (java)
   CGroup: /system.slice/zookeeper.service
           └─1138 java -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -cp /opt/mesosphere/zookeeper/bin/../build/classes:/opt/mesosphere/zookeeper/bin/../build/lib/*.jar:/opt/m...

May 31 18:32:51 d1p3920-charles-mesos-master.vchslabs.vmware.com zookeeper[1138]: at sun.nio.ch.SelectionKeyImpl.interestOps(SelectionKeyImpl.java:77)
```
Make sure ZooKeeper service listens on the port 2181
```
$ netstat -anp | grep 2181
tcp6       0      0 :::2181                 :::*                    LISTEN      27778/java
```
## Mesos
### Extra config for Mesos
Mesos Master IP
```
$ ifconfig eth0 | grep 'inet '
inet 10.145.6.64  netmask 255.255.255.0  broadcast 10.145.6.255

$ echo "10.145.6.64" > /etc/mesos-master/ip
```
Mesos Master Hostname
```
$ nslookup 10.145.6.64
Server:		10.132.71.1
Address:	10.132.71.1#53

64.6.145.10.in-addr.arpa	name = d1p3920-charles-mesos-master.vchslabs.vmware.com.

$ echo "d1p3920-charles-mesos-master.vchslabs.vmware.com" > /etc/mesos-master/hostname
```
Cluster name
```
echo "charles-cluster" > /etc/mesos-master/cluster
```
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
### Start Mesos Master 
Restart the Mesos Master service
```
$ systemctl restart  mesos-master.service
```
Test Mesos Master service
```
$ ps -aux | grep mesos-master
root  2395  /usr/sbin/mesos-master --zk=zk://10.145.6.64:2181/mesos --port=5050 --log_dir=/var/log/mesos --cluster=charles-cluster --hostname=d1p3920-charles-mesos-master.vchslabs.vmware.com. --ip=10.145.6.64 --quorum=1 --work_dir=/var/lib/mesos

$ systemctl status mesos-master.service
   Loaded: loaded (/usr/lib/systemd/system/mesos-master.service; enabled)
   Active: active (running) since Sun 2015-05-31 18:32:34 PDT; 24min ago
 Main PID: 2395 (mesos-master)
   CGroup: /system.slice/mesos-master.service
           ├─2395 /usr/sbin/mesos-master --zk=zk://10.145.6.64:2181/mesos --port=5050 --log_dir=/var/log/mesos --cluster=charles-cluster --hostname=d1p3920-charles-mesos-master.vchslabs.vm...
           ├─2411 logger -p user.info -t mesos-master[2395]
           └─2412 logger -p user.err -t mesos-master[2395]

May 31 18:57:10 d1p3920-charles-mesos-master.vchslabs.vmware.com mesos-master[2412]: I0531 18:57:10.816542  2420 master.cpp:2273] Processing ACCEPT call for offers: [ 20150531-183234-10741...
```
## Marathon
### Start Marathon service
```
$ systemctl restart marathon.service
```
### Test Marathon service
```
$ ps -aux | grep marathon
root  java -Djava.library.path=/usr/local/lib:/usr/lib:/usr/lib64 -Djava.util.logging.SimpleFormatter.format=%2$s%5$s%6$s%n -Xmx512m -cp /usr/bin/marathon mesosphere.marathon.Main --zk zk://10.145.6.64:2181/marathon --master zk://10.145.6.64:2181/mesos

$ systemctl status marathon.service
marathon.service - Marathon
   Loaded: loaded (/usr/lib/systemd/system/marathon.service; enabled)
   Active: active (running) since Sun 2015-05-31 18:26:57 PDT; 32min ago
 Main PID: 1140 (java)
   CGroup: /system.slice/marathon.service
           ├─1140 java -Djava.library.path=/usr/local/lib:/usr/lib:/usr/lib64 -Djava.util.logging.SimpleFormatter.format=%2$s%5$s%6$s%n -Xmx512m -cp /usr/bin/marathon mesosphere.marathon.M...
           ├─1199 logger -p user.info -t marathon[1140]
           └─1200 logger -p user.notice -t marathon[1140]

May 31 18:58:53 d1p3920-charles-mesos-master.vchslabs.vmware.com marathon[1199]: [2015-05-31 18:58:53,360] INFO 10.113.229.247 -  -  [01/Jun/2015:01:58:53 +0000] "GET /v2/apps//hello-marat...
```

## Test the Mesos Master node
### Mesos Master console - port 5050
There is no Mesos Slave node registered so far...
![mesos master console](https://github.com/craimbert/mesos-tutorial/blob/master/screenshots/0-mesos_master_console_5050.png)
### Mesos Master Marathon console - port 8080
![mesos master marathon console](https://github.com/craimbert/mesos-tutorial/blob/master/screenshots/1-marathon_console_8080.png)


# Install 2nd VM -> Mesos Slave node

## Install packages
Add yum repo
```
$ rpm -Uvh http://repos.mesosphere.io/el/7/noarch/RPMS/mesosphere-el-repo-7-1.noarch.rpm
```
Install Mesos & telnet (for testing ports)
```
$ yum -y install mesos telnet
```
## Test network connection with Mesos Master node
Ping the VM at its IP address
```
$ ping 10.145.6.64
PING 10.145.6.64 (10.145.6.64) 56(84) bytes of data.
64 bytes from 10.145.6.64: icmp_seq=1 ttl=64 time=0.979 ms
64 bytes from 10.145.6.64: icmp_seq=2 ttl=64 time=0.456 ms
....
```
Test if the port 2181, used by ZooKeeper on the Mesos Master node, is open
```
$ telnet 10.145.6.64 2181 
Trying 10.145.6.64...
Connected to 10.145.6.64.
Escape character is '^]'.
;
Connection closed by foreign host.
```
## Mesos
### Extra config for Mesos
Mesos Slave IP
```
$ ifconfig eth0 | grep 'inet '
inet 10.145.6.68  netmask 255.255.255.0  broadcast 10.145.6.255

$ echo "10.145.6.68" > /etc/mesos-slave/ip
```
Mesos Slave Hostname
```
$ nslookup 10.145.6.68
Server:		10.132.71.1
Address:	10.132.71.1#53

68.6.145.10.in-addr.arpa	name = d1p3920-charles-mesos-slave.vchslabs.vmware.com.

$ echo "d1p3920-charles-mesos-slave.vchslabs.vmware.com" > /etc/mesos-slave/hostname
```
ZooKeeper list of Master's IP
```
echo "zk://MESOS_MASTER_IP:2181/mesos" > /etc/mesos/zk
```

### Disable mesos-master service
```
$ systemctl stop mesos-master.service
$ systemctl disable mesos-master.service
rm '/etc/systemd/system/multi-user.target.wants/mesos-master.service'
```
### Start Mesos Slave service
Start Mesos Slave service
```
$ systemctl restart mesos-slave.service
```
Test Mesos Master service
```
$ ps -aux | grep mesos-slave
root  /usr/sbin/mesos-slave --master=zk://10.145.6.64:2181/mesos --log_dir=/var/log/mesos --hostname=d1p3920-charles-mesos-slave.vchslabs.vmware.com. --ip=10.145.6.68

$ systemctl status mesos-slave.service
systemctl status mesos-slave.service
mesos-slave.service - Mesos Slave
   Loaded: loaded (/usr/lib/systemd/system/mesos-slave.service; enabled)
   Active: active (running) since Sun 2015-05-31 18:34:06 PDT; 38min ago
 Main PID: 32373 (mesos-slave)
   CGroup: /system.slice/mesos-slave.service
           ├─32373 /usr/sbin/mesos-slave --master=zk://10.145.6.64:2181/mesos --log_dir=/var/log/mesos --hostname=d1p3920-charles-mesos-slave.vchslabs.vmware.com. --ip=10.145.6.68
           ├─32383 logger -p user.info -t mesos-slave[32373]
           └─32384 logger -p user.err -t mesos-slave[32373]

May 31 19:02:07 localhost.localdomain mesos-slave[32384]: I0531 19:02:07.241608 32385 slave.cpp:3648] Current disk usage 6.79%. Max allowed age: 5.824677064796389days
```
## Test the Mesos Slave node
A slave node is registered into the Mesos Master
![mesos slave 0](https://github.com/craimbert/mesos-tutorial/blob/master/screenshots/2-mesos_master_console_new_slave.png)
<br />
The 1rst node appears in the list of registered nodes on the master
![mesos slave 1](https://github.com/craimbert/mesos-tutorial/blob/master/screenshots/3-mesos_master_console_list_slave.png)
<br />
Here is the summary of the 1rst Mesos Slave node
![mesos slave 2](https://github.com/craimbert/mesos-tutorial/blob/master/screenshots/4-mesos_master_console_slave_summary.png)
<br />

# Tests the Cluster

## Connection Mesos Slave -> Mesos Master
```
$ mesos-resolve `cat /etc/mesos/zk`
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@712: Client environment:zookeeper.version=zookeeper C client 3.4.5
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@716: Client environment:host.name=localhost.localdomain
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@723: Client environment:os.name=Linux
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@724: Client environment:os.arch=3.10.0-123.el7.x86_64
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@725: Client environment:os.version=#1 SMP Mon Jun 30 12:09:22 UTC 2014
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@733: Client environment:user.name=root
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@741: Client environment:user.home=/root
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@log_env@753: Client environment:user.dir=/etc/mesos-slave
2015-05-31 19:14:28,941:32455(0x7f0c8a303700):ZOO_INFO@zookeeper_init@786: Initiating client connection, host=10.145.6.64:2181 sessionTimeout=10000 watcher=0x7f0c9155f1e0 sessionId=0 sessionPasswd=<null> context=0x7f0c74001160 flags=0
2015-05-31 19:14:28,942:32455(0x7f0c858ee700):ZOO_INFO@check_events@1703: initiated connection to server [10.145.6.64:2181]
2015-05-31 19:14:28,956:32455(0x7f0c858ee700):ZOO_INFO@check_events@1750: session establishment complete on server [10.145.6.64:2181], sessionId=0x14dacbaa0b80013, negotiated timeout=10000
WARNING: Logging before InitGoogleLogging() is written to STDERR
I0531 19:14:28.956903 32463 group.cpp:313] Group process (group(1)@127.0.0.1:49647) connected to ZooKeeper
I0531 19:14:28.956985 32463 group.cpp:790] Syncing group operations: queue size (joins, cancels, datas) = (0, 0, 0)
I0531 19:14:28.957010 32463 group.cpp:385] Trying to create path '/mesos' in ZooKeeper
I0531 19:14:28.959980 32463 detector.cpp:138] Detected a new leader: (id='9')
I0531 19:14:28.960134 32463 group.cpp:659] Trying to get '/mesos/info_0000000009' in ZooKeeper
I0531 19:14:28.961226 32463 detector.cpp:452] A new leading master (UPID=master@10.145.6.64:5050) is detected
10.145.6.64:5050
```

## Launch task from Mesos Slave node
Best way to test: launch a task through mesos-execute from mesos-slave node

### Set the MASTER
```
$ export MASTER=$(mesos-resolve `cat /etc/mesos/zk`)
$ echo $MASTER
10.145.6.64:5050
```

### Launch the task
```
$ mesos-execute --master=$MASTER --name="cluster-test" --command="sleep 5"

I0531 19:15:50.273203 32492 sched.cpp:157] Version: 0.22.1
I0531 19:15:50.277058 32497 sched.cpp:254] New master detected at master@10.145.6.64:5050
I0531 19:15:50.277282 32497 sched.cpp:264] No credentials provided. Attempting to register without authentication
I0531 19:15:50.279747 32497 sched.cpp:448] Framework registered with 20150531-183234-1074172170-5050-2395-0003
Framework registered with 20150531-183234-1074172170-5050-2395-0003
task cluster-test submitted to slave 20150531-183234-1074172170-5050-2395-S0
Received status update TASK_RUNNING for task cluster-test
Received status update TASK_FINISHED for task cluster-test
I0531 19:15:55.405704 32496 sched.cpp:1589] Asked to stop the driver
I0531 19:15:55.405743 32496 sched.cpp:831] Stopping framework '20150531-183234-1074172170-5050-2395-0003'
```
### Result in the Master Slave console
Under Slaves / Completed Frameworks, the list Completed Executors presents the list of executed task


# Recurrent Problems
## LIBPROCESS_IP not defined for the Mesos Slave node
Problem
```
$ mesos-execute --master=$MASTER --name="cluster-test" --command="sleep 5"
**************************************************
Scheduler driver bound to loopback interface! Cannot communicate with remote master(s). You might want to set 'LIBPROCESS_IP' environment variable to use a routable IP address.
**************************************************
```
Solution: set LIBPROCESS_IP as an env variable
```
$ export LIBPROCESS_IP=10.145.6.68
```
## Issues coming from previous mesos runs
Clear the cache saved from prior run
```
$ systemctl stop mesos-slave.service
$ rm -f /tmp/mesos/meta/slaves/latest
$ systemctl start mesos-slave.service
$ systemctl status mesos-slave.service
```
## iptables
The iptables on a CentOS 7 VM should look like this
```
$ iptables -L

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
