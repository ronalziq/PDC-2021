<div id="top"></div>



<!-- PROJECT LOGO -->
<br />
<div align="center">

  <h3 align="center">HA Cluster</h3>

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
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contributing">Contributing</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgments">Acknowledgments</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project

[![Product Name Screen Shot][product-screenshot]](https://example.com)

First we have created 2 nodes after that we have assigned ips to nano file to connec tboth the nodes together.


This file is used to resolve a name into an address
```sh
nano /etc/hosts

```
To show complete network details.

```sh
ip a
```
Displays Current hostname
```sh
hostname
```
It will change the name of hostname
```sh
hostname nodel

```
It helps to change the hostname without actually locating and editing the /etc/hostname file on a given system.

```sh

hostnamectl
```
Installing this package for your Enterprise Linux version should allow you to use normal tools such as yum to install packages and their dependencies
```sh
dnf install epel-release
```
It will help in installing chrony to help in flexible implementation of the Network Time Protocol (NTP).
```sh
yum -y install chrony
```
It will open the config file that is stored in etc
```sh
nano /etc/chrony.conf
```
It will retart the chrony service for changes to be applied.
```sh
systemctl restart chronyd
```
It will enable chrony 
```sh
systemctl enable chronyd
```
It is a firewall command that helps us to add ntp service.
```sh
firewall-cmd -permanent --add-service=ntp
```
It will help firewall command to reload
```sh
firewall-cmd --reload
```

## Step 2: Install pacemaker and other High Availability rpms

By default the High Availability repository is disabled on CentOS 8 based on the below commit
```sh
2019-12-19 - bstinson@centosproject.org - 8-1.0.7
- Typo fixes
- Disable the HA repo by default
```

So before we install CentOS HA Cluster rpms, we will enable the HighAvailability repo
```sh
[root@centos8-1 ~]# dnf config-manager --set-enabled HighAvailability
```

List the enabled repos
```sh
[root@centos8-1 ~]# dnf repolist
Last metadata expiration check: 0:00:07 ago on Sun 05 Apr 2020 01:40:56 PM IST.
repo id                          repo name                                                               status
AppStream                        CentOS-8 - AppStream                                                    5,124
BaseOS                           CentOS-8 - Base                                                         2,126
HighAvailability                 CentOS-8 - HA                                                             130
PowerTools                       CentOS-8 - PowerTools                                                   1,525
*epel                            Extra Packages for Enterprise Linux 8 - x86_64                          5,138
*epel-modular                    Extra Packages for Enterprise Linux Modular 8 - x86_64                      0
extras                           CentOS-8 - Extras                                                          12
```
Install pacemaker Linux and other high availability rpms using DNF which is the default package manager in RHEL/CentOS 8.
```sh
[root@centos8-1 ~]# dnf install pcs pacemaker fence-agents-all -y
```
## Step 3: Start pacemaker cluster manager service
Before we configure Linux HA Cluster, the pcs daemon must be started and enabled to start at boot time on each node of the Linux HA Cluster. This daemon works with the pcs command-line interface to manage synchronizing the corosync configuration across all nodes in the cluster.
```sh
[root@centos8-1 ~]# systemctl enable pcsd.service --now
Created symlink /etc/systemd/system/multi-user.target.wants/pcsd.service → /usr/lib/systemd/system/pcsd.service.
```
## Step 4: Assign password to hacluster
hacluster user is created after installing high availability cluster rpms with disabled password. Set a password for user hacluster on each node in the Linux HA cluster and authenticate user hacluster for each node in the cluster on the node from which you will be running the pcs commands.
```sh
[root@centos8-1 ~]# passwd hacluster
Changing password for user hacluster.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```
## Step 5: Configure firewalld
If you are running the firewalld daemon, enable the ports that are required by the CentOS High Availability repo on all the Cluster nodes.
```sh
[root@centos8-1 ~]# firewall-cmd --permanent --add-service=high-availability
success

[root@centos8-1 ~]# firewall-cmd --reload
success
```
## Step 6: Configure Corosync
On any one of the cluster node, use pcs host auth to authenticate as the hacluster user. Use the below syntax:
```sh
pcs host auth [node1] [node2] [node3] ..
```

With RHEL/CentOS 7 High Availability Cluster with pacemaker Linux, we used pcs cluster auth to authenticate the clusters but this has changed with RHEL/CentOS 8 to "pcs host auth"
```sh
[root@centos8-1 ~]# pcs host auth centos8-1.example.com centos8-2.example.com centos8-3.example.com
Username: hacluster
Password:
centos8-2.example.com: Authorized
centos8-3.example.com: Authorized
centos8-1.example.com: Authorized
```
Now since the cluster nodes are authorized so we can proceed with the next step to setup high availability cluster. Here I am creating a three node Linux HA cluster with the name "my_cluster"
```sh
No addresses specified for host 'centos8-1.example.com', using 'centos8-1.example.com'
No addresses specified for host 'centos8-2.example.com', using 'centos8-2.example.com'
No addresses specified for host 'centos8-3.example.com', using 'centos8-3.example.com'
Destroying cluster on hosts: 'centos8-1.example.com', 'centos8-2.example.com', 'centos8-3.example.com'...
centos8-1.example.com: Successfully destroyed cluster
centos8-3.example.com: Successfully destroyed cluster
centos8-2.example.com: Successfully destroyed cluster
Requesting remove 'pcsd settings' from 'centos8-1.example.com', 'centos8-2.example.com', 'centos8-3.example.com'
centos8-1.example.com: successful removal of the file 'pcsd settings'
centos8-3.example.com: successful removal of the file 'pcsd settings'
centos8-2.example.com: successful removal of the file 'pcsd settings'
Sending 'corosync authkey', 'pacemaker authkey' to 'centos8-1.example.com', 'centos8-2.example.com', 'centos8-3.example.com'
centos8-1.example.com: successful distribution of the file 'corosync authkey'
centos8-1.example.com: successful distribution of the file 'pacemaker authkey'
centos8-2.example.com: successful distribution of the file 'corosync authkey'
centos8-2.example.com: successful distribution of the file 'pacemaker authkey'
centos8-3.example.com: successful distribution of the file 'corosync authkey'
centos8-3.example.com: successful distribution of the file 'pacemaker authkey'
Sending 'corosync.conf' to 'centos8-1.example.com', 'centos8-2.example.com', 'centos8-3.example.com'
centos8-1.example.com: successful distribution of the file 'corosync.conf'
centos8-3.example.com: successful distribution of the file 'corosync.conf'
centos8-2.example.com: successful distribution of the file 'corosync.conf'
Cluster has been successfully set up.
```
## Step 7: Start and Verify Cluster
## Step 7.1: Start Linux HA Cluster
After configering corosync, let's start the cluster. The command below will start corosync and pacemaker Linux on all our nodes in the cluster.
```sh
[root@centos8-1 ~]# pcs cluster start --all
```
## Step 7.2: Verify corosync installation
corosync-cfgtool is a tool for displaying and configuring active parameters within corosync
```sh
[root@centos8-1 ~]# corosync-cfgtool -s
Printing link status.
Local node ID 1
LINK ID 0
        addr    = 10.10.10.12
        status:
                nodeid  1:      link enabled:1  link connected:1
                nodeid  2:      link enabled:1  link connected:1
                nodeid  3:      link enabled:1  link connected:1
```
Here
```sh
  Displays  the  status  of the current links on this node for UDP/UDPU, with extended status for KNET. 
        If any interfaces are faulty, 1 is returned by the binary. If all interfaces are active 0 is returned to the shell.
```
We can see here that everything appears normal with our fixed IP address (not a 127.0.0.x loopback address) listed as the id, and no faults for the status.
If you see something different, you might want to start by checking the node’s network, firewall and SELinux configurations.

Next, check the membership and quorum APIs:
```sh
[root@centos8-1 ~]# corosync-cmapctl | grep members
runtime.members.1.config_version (u64) = 0
runtime.members.1.ip (str) = r(0) ip(10.10.10.12)
runtime.members.1.join_count (u32) = 1
runtime.members.1.status (str) = joined
runtime.members.2.config_version (u64) = 0
runtime.members.2.ip (str) = r(0) ip(10.10.10.16)
runtime.members.2.join_count (u32) = 3
runtime.members.2.status (str) = joined
runtime.members.3.config_version (u64) = 0
runtime.members.3.ip (str) = r(0) ip(10.10.10.17)
runtime.members.3.join_count (u32) = 3
runtime.members.3.status (str) = joined
```
Check the status of corosync across cluster nodes
```sh
[root@centos8-1 ~]# pcs status corosync

Membership information
----------------------
    Nodeid      Votes Name
         1          1 centos8-1.example.com (local)
         2          1 centos8-2.example.com
         3          1 centos8-3.example.com
```
## Step 7.3: Verify Pacemaker Linux Installation
Now that we have confirmed that Corosync is functional, we can check the rest of the stack. Pacemaker Linux has already been started.
```sh
[root@centos8-1 ~]# pcs status
Cluster name: my_cluster

WARNINGS:
No stonith devices and stonith-enabled is not false

Stack: corosync
Current DC: centos8-1.example.com (version 2.0.2-3.el8_1.2-744a30d655) - partition with quorum
Last updated: Sun Apr  5 15:25:03 2020
Last change: Sun Apr  5 15:24:23 2020 by hacluster via crmd on centos8-1.example.com

3 nodes configured
0 resources configured

Online: [ centos8-1.example.com centos8-2.example.com centos8-3.example.com ]

No resources


Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/enabled
```
We will also use corosync and pacemaker service to auto start on boot on all the Linux HA Cluster nodes

```sh
[root@centos8-1 ~]# systemctl enable corosync
Created symlink /etc/systemd/system/multi-user.target.wants/corosync.service → /usr/lib/systemd/system/corosync.service.

[root@centos8-1 ~]# systemctl enable pacemaker
Created symlink /etc/systemd/system/multi-user.target.wants/pacemaker.service → /usr/lib/systemd/system/pacemaker.service.
```
## Step 8: Disable Fencing (Optional)
A High Availability Cluster requires you to configure fencing for the cluster to control the cluster nodes. Fencing is known as STONITH, an acronym for "Shoot The Other Node In The Head", since the most popular form of fencing is cutting a host’s power.

```sh
[root@centos8-1 ~]# pcs property set stonith-enabled=false
```