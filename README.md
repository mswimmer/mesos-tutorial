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
Install Mesos & Marathon
```
$ yum -y install mesos marathon
```
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

### Install packages
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
### Extra config for Mesos

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
### Restart Mesos Slave
```
$ systemctl restart mesos-slave.service
$ systemctl status mesos-slave.service
mesos-slave.service - Mesos Slave
   Loaded: loaded (/usr/lib/systemd/system/mesos-slave.service; enabled)
   Active: active (running) since Sat 2015-05-30 19:41:47 PDT; 6s ago
 Main PID: 2784 (mesos-slave)
   CGroup: /system.slice/mesos-slave.service
           ├─2784 /usr/sbin/mesos-slave --master=zk://10.145.6.64:2181/mesos --log_dir=/var/log/mesos
           ├─2790 logger -p user.info -t mesos-slave[2784]
           └─2791 logger -p user.err -t mesos-slave[2784]

May 30 19:41:47 localhost.localdomain mesos-slave[2791]: I0530 19:41:47.750658  2792 group.cpp:313] Group process (group(1)@127.0.0....Keeper
May 30 19:41:47 localhost.localdomain mesos-slave[2791]: I0530 19:41:47.750701  2792 group.cpp:790] Syncing group operations: queue ... 0, 0)
```
# Test connection Mesos Slave -> Mesos Master
```
$ mesos-resolve `cat /etc/mesos/zk`
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@712: Client environment:zookeeper.version=zookeeper C client 3.4.5
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@716: Client environment:host.name=localhost.localdomain
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@723: Client environment:os.name=Linux
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@724: Client environment:os.arch=3.10.0-123.el7.x86_64
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@725: Client environment:os.version=#1 SMP Mon Jun 30 12:09:22 UTC 2014
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@733: Client environment:user.name=root
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@741: Client environment:user.home=/root
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@log_env@753: Client environment:user.dir=/root
2015-05-30 19:43:37,365:2819(0x7f1fb4822700):ZOO_INFO@zookeeper_init@786: Initiating client connection, host=10.145.6.64:2181 sessionTimeout=10000 watcher=0x7f1fbba7e1e0 sessionId=0 sessionPasswd=<null> context=0x7f1fa4003270 flags=0
2015-05-30 19:43:37,366:2819(0x7f1fabdf3700):ZOO_INFO@check_events@1703: initiated connection to server [10.145.6.64:2181]
2015-05-30 19:43:37,372:2819(0x7f1fabdf3700):ZOO_INFO@check_events@1750: session establishment complete on server [10.145.6.64:2181], sessionId=0x14da7b7ef3e000f, negotiated timeout=10000
WARNING: Logging before InitGoogleLogging() is written to STDERR
I0530 19:43:37.372766  2821 group.cpp:313] Group process (group(1)@127.0.0.1:50469) connected to ZooKeeper
I0530 19:43:37.372812  2821 group.cpp:790] Syncing group operations: queue size (joins, cancels, datas) = (0, 0, 0)
I0530 19:43:37.372824  2821 group.cpp:385] Trying to create path '/mesos' in ZooKeeper
I0530 19:43:37.374236  2821 detector.cpp:138] Detected a new leader: (id='2')
I0530 19:43:37.374328  2821 group.cpp:659] Trying to get '/mesos/info_0000000002' in ZooKeeper
I0530 19:43:37.374979  2821 detector.cpp:452] A new leading master (UPID=master@10.145.6.64:5050) is detected
10.145.6.64:5050
```

# Test
## Mesos Master console - port 5050

## Mesos Master Marathon console - port 8080
