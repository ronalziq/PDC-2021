<div id="top"></div>




<br />
<div align="center">
 

  <h3 align="center">High Availability Cluster</h3>

  <p align="center">
   Built on Linux Centos 8 distribution!
   By Group A-2
    <br />
        <br />
    <br />
  </p>
</div>




<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#infrastructure">Infrastructure</a>
    </li>
    <li><a href="#cluster">Cluster</a></li>
    <li><a href="groupmembers">Group Members</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

HA cluster is also known as failover cluster or active-passive cluster. This type of cluster is one of the most widely used in a production environment maintain continuous availability of services even if one of the node in cluster fails. Technically, if the server running application has failed for any reason so the pacemaker (cluster software) will reboot the application on the working node. Failover is not only rebooting an application, it is infact a series of operations linked with it, like mounting filesystems, configuring networks, and starting dependent applications.

<p align="right">(<a href="#top">back to top</a>)</p>



### Built With

We have used these services/platform to make this High availability cluster.

* [Centos 8]
* [Apache]
* [Pacemaker]
* [Heartbeat]
* [Corony]


<p align="right">(<a href="#top">back to top</a>)</p>



<!-- GETTING STARTED -->
## Infrastruture of HA Cluster

We are using 4 machines:

* 20GB dedicated to each machine and one extra 10GB disk to server for shared storage.
* Hostname 		IP Address		Operating System	Purpose
* Node1.local		192.168.218.133		Centos 8		Cluster Node 1
* Node2.local		192.168.218.135		Centos 8		Cluster Node 1
* Server			192.168.218.134		Centos 8		Shared Storage between Node1&Node2
* Virtual IP		192.168.218.136		Centos 8		Virtual Cluster IP(Host web app)

### Prerequisites

Login all machines from root and install targetcli and LVM2 packages on both NODE 1 & NODE 2.
* Install targetcli and lvm2 packages in server machine
  ```sh
  yum install -y targetcli lvm2
  ```
* Install iscsi initiator and lvm2 packages on both node1 and node2.
  ```sh
  yum install -y iscsi-initiator-utils lvm2
  ```


<!-- USAGE EXAMPLES -->
## Usage

. To setup the shared storage disk
   ```sh
  fdisk -l
  pvcreate /dev/---
  vgcreate vg_iscsi /dev/---
  lvcreate -l 100%FREE -n lv_iscsi vg_iscsi
   ```
. To generate initiator name on both machine (node1 & node2).
   ```sh
   cat /etc/iscsi/initiatorname.iscsi
   ```
. To enter the commandline of iscsi in block,: 
  ```sh
   cd /iscsi
   cd /backstores/block
   create iscsi_shared_storage /dev/vg_iscsi/lv_iscsi
   ```
. move to iscsi, 
  ```sh
   targetcli
   create
   ```
. move to acls in tpg1 in target iqn. 
  ```sh
   cd iqn.2003-01.org.linux-iscsi.local.x8664:sn.48f061372e0c/tpg1/acls
   ```
   in both nodes 
   ```sh
   create iqn.1994-05.com.redhat:e9d88fe8270
   ```
   
 . then we have to go inside luns (Logical Unit Number)
  ```sh
   cd /iscsi/iqn.2003-01.org.linux-iscsi.local.x8664:sn.48f061372e0c/tpg1/luns
   ```
   create LUN 
   ```sh
   create /backstores/block/iscsi_shared_storage
   ```
 . then save the configuration and exit cli of iscsi 
  ```sh
   cd /
   ls
   saveconfig
   exit
   ```
   
 .  Restart and enable the target
   ```sh
   systemctl restart target
   systemctl enable target
   ```
   
  . Configure the firewall to allow iscsi
 ```sh
  firewall-cmd --permanent --add-port=3260/tcp
  firewall-cmd --reload
   ```
. In node1:
Using the IP address of server
execute below command on both machine node1 and node2
```sh
iscsiadm -m discovery -t st -p 192.168.218.134
```
It will discover the lun on both machines.
Now, we will map the lun on both nodes
```sh
iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.local.x8664:sn.48f061372e0c 192.168.218.134.
```
Lun will be mapped.
After using on both nodes
```sh
lsblk
```
It will confirmed that 10GB disk of shared storage is now visible on both node1 and node2.

Now we have to create lvm by using either node1 or node2:
In node1:
```sh
 pvcreate /dev/---
 vgcreate vg_apache /dev/---
 lvcreate -n lv_apache -l 100%FREE vg_apache
```
Then make file system:
```sh
 mkfs.ext4 /dev/vg_apache/lv_apache
```
We confirm this physical volumer and lvm on node2 by using:
```sh
 pvscan
 vgscan
 lvscan
```
* Make entry of both nodes in hosts file
Open hosts editor on node1:
```sh
 vim /etc/hosts
```
Insert hostnames and respective IP addresses.
Save and quit editor.
**Perform same on node2**
Cluster packages are available in ha repositary.
Enable these on both nodes:
```sh
yum config-manager -set-enable ha
```
After this install below package on both nodes:
```sh
yum install -y pcs fence-agents-all pcp-zeroconf
```

Now add the rule for high availabilitytraffic in firewall.
In node1
Execute firewall:
```sh
 firewall-cmd --permanent --add-service=high-availability
 firewall-cmd --add-service=high-availability 
 firewall-cmd --reload
```

Set the password for hacluster: **Perform same on Node2**
```sh
 passwd hacluster
 12345678
 12345678
```
Start and enable the pcsd services
```sh
 systemctl start pcsd
 systemctl enable pcsd
```
In node1
We will authorize both nodes
```sh
 pcs host auth node1.local node2.local
username: hacluster
password: 12345678
```
**Output**
node1.local: Authorized
node2.local: Authorized
**Now we will create the cluster**
Execute:
```sh
 pcs cluster setup local_cluster --start node1.local node2.local
```
Cluster will be successfully setup here.
Now we will set up the cluster to start automatically on a startup of the machine.
Execute:
```sh
= pcs cluster enable --all
```
**Output**
node1.local: Cluster Enabled 
node2.local: Cluster Enabled
To check the status of cluster
Use:
```sh
 pcs cluster status
```
To get detailed Info about cluster:
```sh
 pcs status
```

Now install apache on both the nodes
```sh
 yum install -y httpd
```
In node1 and node2:
Open httpd configuration
```sh
 vim /etc/httpd/conf/httpd.conf
```
go at the last line and add these lines:
```sh
<location> /server-status>
	SetHandler server-status
	Required local
</location>
```
Save and quite the editor.
Now, we will edit the apache web server's logrotate configuration to tell not to use systemd as cluster resource and does not use systemd to reload the service.
In both node1 and node2
Execute:
```sh
 /bin/systemctl reload reload httpd.service > /dev/null 2>/dev/null || true
 /usr/sbin/httpd -f /etc/httpd/conf/httpd.conf -c "PidFile /var/run/httpd.pid" -k graceful > /dev/null 2>/dev/null || true;
```
Now, we will use the shared storage for storing the web content that is the html file and we will perform this operation in any of the node.
In node1:
```sh
 mount /dev/vg_apache/lv_apache /var/www
 mkdir /var/www/html
 mkdir /var/www/cgi-bin
 mkdir /var/www/error
```
Execute:
```sh
 restorecon -R /var/www
 cat <<-END >/var/www/html/index.html
 unmount /var/www
```
Allow apache service in the firewall on both the node:
On node1 and node2
Execute:
```sh
 firewall-cmd --permanent --add-service=http
 firewall-cmd --reload
 pcs resource create httpd_fs Filesystem device="/dev/mapper/vg_apache-lv_apache" directory="/var/www" fstype="ext4" --group apache;
```
<p align="right">(<a href="#top">back to top</a>)</p>

**Create IP address resource (virtual IP)**
```sh
 ip a
```
192.168.218.136
use IP of virtual IP in node1 and node2
**************************************************************************************************************
On node1 **Create an apache resource to monitor the status of apache server that will move resource to another node just in case of any type of failure**
Execute:
```sh
 pcs resource create httpd_vip IPaddr2 ip=192.168.218.136 cidr_netmask=24 --group apache
 pcs resource create httpd_ser apache configfile="/etc/httpd/conf/httpd.conf" statusurl="http://127.0.0.1/server-status" --group apache;
```


<!-- ROADMAP -->
## Cluster

********************ALL MACHINES ARE READY, ClUSTER IS READY, AND WEB SERVER IS READY TO USE*****************
To access cluster:
Enter Virtual IP address in browser
To standby any node
Execute:
```sh
 pcs node standby node1.local
```
To restart resource
Execute:
```sh
 pcs resource restart <resource_name>;
 pcs resource move <resource_group> <destination> 
 pcs property set mainteinance-mode=true  
 pcs property set mainteinance-mode=false
 pcs cluster start <node_name>
 pcs cluster stop <node_name>
 pcs cluster start --all
 pcs cluster stop --all
 pcs cluster destroy <cluster_name>
```
<p align="right">(<a href="#top">back to top</a>)</p>



<!-- CONTRIBUTING -->
## groupmembers


1. Daniyal Malik 
2. Rashwan 
3. Muhammad Obaid
4. Rohan Kukreja

<p align="right">(<a href="#top">back to top</a>)</p>
