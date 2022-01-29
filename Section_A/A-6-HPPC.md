# HPC Cluster readme

##Group Members
Izhar Karim Baig - 1812115
Nabeel- 1812126
Muhammad Shakir - 1812132


## Setting Up virtual machines

We setup 3 virtual machines having Centos 8 on each Nodes, The first Node is master node which is named as 'masternode' and the other 2 will be computing nodes named as 'HPCNode1' and 'HPCNode2'.


### Host File Configuration

we configured hosts file with the IP addresses of the respective nodes on every node.

1. Open the /etc/hosts file 

```
	nano /etc/hosts
```

2. we inserted the following entries in the hosts file 

```
	192.168.128.120 masternode
	192.168.128.121 HPCNode1
	192.168.128.122 HPCNode2
```

###Establishing SSH

1.We created two key pairs in the masternode in  /root/.ssh/ directory. 

```
	ssh-keygen -t dsa;
	ssh-heygen -t rsa;

```

2. After that we merged the  RSA and DSA public key to the authorized_keys file.

3. Now we will duplicate the /root/.ssh/ directory to HPCNode1 and HPCNode2. 

```
	scp -r .ssh/ HPCNode1:/root
	scp -r .ssh/ HPCNode2:/root



```

4. next we  will create SSH fingerprints for every node to avoid credentials

```
	ssh-keyscan -t dsa masternode HPCNode1 HPCNode2 >> /etc/ssh/ssh_known_hosts;
	ssh-keyscan -t rsa masternode HPCNode1 HPCNode2 >> /etc/ssh/ssh_known_hosts;

```

5. /etc/ssh/ssh_known_hosts  file is copied to HPCNode1 and HPCNode2. 

### Setting Up and Configuring NTP

1. we will be setting Up NTP server on masternode in /etc/chrony.conf file. 

```


	#server 0.rhel.pool.ntp.org iburst
	#server 1.rhel.pool.ntp.org iburst
	#server 2.rhel.pool.ntp.org iburst
	#server 3.rhel.pool.ntp.org iburst
	server 192.168.64.128 prefer iburst

# save the rate at which the system clock gains and losses time.

```
driftfile /var/lib/chrony/drift
local stratum 8
manual

```

# enable the system clock to be stepped in the first three updates

makestep 1.0 3(greater than 1 second)

# activate kernel synchronization  RTC.

rtcsync

# Enabling hardware timestamp on all interfaces.

#hwtimestamp *

#   the minimum number of selectable sources required to adjust

# the system clock.
#minsources 2

# enabling NTP client access from local network.

#allow 192.168.128.0/16

allow 192.168.128.0/24

# Serve time even if synchronization is not performed.

#local stratum 10

# defining file containing keys for NTP authentication.

#keyfile /etc/chrony.keys

# defining directory for log files.

logdir /var/log/chrony

# Selecting the data the which  is logged.
#log measurements statistics tracking
```

2. Saving  the changes in /etc/chrony.conf file.

3. restarting the chronyd service.

```
systemctl restart chronyd;
 
chronyc makestep;
chronyc ntpdata;
timedatectl;

```
ep;
chronyc ntpdata;
timedatectl;

```





