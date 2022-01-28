# HPC Cluster Guide

## Pre-HPC Setup
Setup three virtual machines. We will be installing CentOS 8 on each machine. The first VM will act as a master node of the HPC cluster. The second and third VM will be the computing nodes. The configuration of the VMs is as follows,
| Virtual Machine | Hostname |IP address|
|-----------------|----------|----------|
|HPC - Master Node|master-node|192.168.64.128|
|HPC - Node - 1 |node-1|192.168.64.129|
|HPC - Node - 2|node-2|192.168.64.130 |

### Configure Hosts File
Configure /etc/hosts file with the IP addresses of the respective nodes on each node.
1. Open the `/etc/hosts` file 
```
nano /etc/hosts
```
2. Add the following entries in the /etc/hosts file 
```
192.168.64.128 master-node
192.168.64.129 node-1
192.168.64.130 node-2
```

### Setting Up SSH
1. In the master-node, Generate two key pairs, first DSA and the second RSA, by running the following commands two key pairs will be created in the `/root/.ssh/` directory. When running the commands just press enter at the prompts so the default settings are applied when keys are being generated. 
```
ssh-keygen -t dsa;
ssh-heygen -t rsa;
```

2. Append the generated RSA and DSA public key to the `/root/.ssh/`authorized_keys file.
```
cat *.pub >> authorized_keys
```

3. Now we will copy the `/root/.ssh/` directory to node-1 and node-2. 
```
scp -r .ssh/ node-1:/root/;
scp -r .ssh/ node-2:/root/;
```

4. We will now generate SSH fingerprints for master-node, node-1 and node-2 so that we do not have to enter the password every time communication is taking place between the nodes. 

5. In the master-node, generate fingerprints for each node and store DSA and RSA fingerprints in the file `/etc/ssh/ssh_known_hosts`. 
```
ssh-keyscan -t dsa master-node node-1 node-2 >> /etc/ssh/ssh_known_hosts;
ssh-keyscan -t rsa master-node node-1 node-2 >> /etc/ssh/ssh_known_hosts;
```

6. Now copy the `/etc/ssh/ssh_known_hosts` file to node-1 and node-2. 
```
scp /etc/ssh/ssh_known_hosts node-1:/etc/ssh/;
scp /etc/ssh/ssh_known_hosts node-2:/etc/ssh/;
```
**Note:Now You will be able to ssh from each node without a password**

### Configure NTP server
Now set up the NTP server/clients using the Chrony utility since the NTP utility has depreciated 
1. On the master-node we will be setting up an NTP server. Add the following configuration to the `/etc/chrony.conf` file. 
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server 192.168.64.128 prefer iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift
local stratum 8
manual

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16
allow 192.168.64.0/24

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```

2. Save the changes in `/etc/chrony.conf` file.

3. Now restart the chronyd service.
```
systemctl restart chronyd;
 
chronyc makestep;
chronyc ntpdata;
timedatectl;
```

4. Set the Chonry service on client machines. Add the following configuration to the `/etc/chrony.conf` file.
```
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
#server 0.rhel.pool.ntp.org iburst
#server 1.rhel.pool.ntp.org iburst
#server 2.rhel.pool.ntp.org iburst
#server 3.rhel.pool.ntp.org iburst
server 192.168.64.128 prefer iburst
 
# Record the rate at which the system clock gains/losses time.
server master iburst
driftfile /var/lib/chrony/drift
logdir /var/log/chrony
log measurements statistics tracking
 
# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3
 
# Enable kernel synchronization of the real-time clock (RTC).
rtcsync
 
# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *
 
# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2
 
# Allow NTP client access from local network.
#allow 192.168.0.0/16
#allow 192.168.64.0/24
 
# Serve time even if not synchronized to a time source.
#local stratum 10
 
# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys
 
# Specify directory for log files.
logdir /var/log/chrony
 
# Select which information is logged.
#log measurements statistics tracking
```

5. Save the changes in the /etc/chrony.conf file.

6. Now restart the chronyd service and sync to the NTP server's clock. 
```
systemctl restart chronyd;
 
chronyc makestep;
chronyc ntpdata;
timedatectl;
```

7. Check the clients are synchronized with the NTP server. if the second letter in the output is '*', it indicates that the client is currently in sync with the server. 
```
chronyc sources
```
### Setup PDSH
We will now install the PDSH utility.

1. Download latest pdsh source file from https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/pdsh/pdsh-2.29.tar.bz2
```
curl -L -O https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/pdsh/pdsh-2.29.tar.bz2
```

2. Extract the pdsh-2.29.tar.bz2 file
```
tar -xf pdsh-2.29.tar.bz2
```

3. Navigate to the `pdsh-2.29` directory 
```
cd pdsh-2.29
```

4.  Execute the configuration file. 
```
 ./configure --with-ssh --without-rsh --with-machines=/etc/machines
```

5. Now make the binary file and link the binaries. 
```
make;
make install;
ln -s /usr/local/bin/pdsh /bin/pdsh;
```

6. Add the cluster nodes to the `/etc/machines` file. 
```
master-node
node-1
node-2
```

### Setup NFS dirctory
Now we will be setting up our NFS directories.
1. Create a `/cluster` directory on all nodes. 
```
 pdsh -a mkdir /cluster -p
```
**Note: `-p` is used because if the directory is already created it will not prompt and will just run the remaining command**

2. Add an entry to the `/etc/exports` file to allow all the nodes to read and write to the `/cluster` directory of the master-node.
```
/cluster        *(rw,no_root_squash,sync)
```

3. Now restart the NFS service and will configure the NFS service to start at boot. 
```
systemctl start nfs-server.service;
systemctl enable nfs-server.service;
```

4. Mount the `/cluster` directory of the master-node to `/cluster` directory of node-1 and node-2.
```
mount -t nfs master-node:/cluster /cluster
```

5. Use the following command to check if the directory has mounted on node-1 and node-2 
```
 df -hT
```

6. Now we will add an entry in the `/etc/fstab` file so the next time node is rebooted it mounts the directory itself. 
```
master-node:/cluster    /cluster                nfs     defaults        0 0
```

### Create User
Set up a user named mpiuser associated with group mpigroup. The user will have id 600, group id will also be 600 and the home directory of the user will be /cluster/mpiuser on all the nodes.  
```
groupadd -g 600 mpigroup;
useradd -u 600 -g 600 -d /cluster/mpiuser mpiuser;
```
We will also add ssh keys for the user as done in **Setting Up SSH** section

### Install Requerd Packages
Install GCC and G77 on all nodes 
```
yum groupinstall "Development Tools" -y;
yum install yum install compat-gcc-34-g77 -y;
```

### Setup MPI
1. Download the MPI source file from https://www.mpich.org/static/downloads/1.0.8/mpich2-1.0.8.tar.gz

2. Extract the mpich2-1.0.8.tar.gz file
```
tar -xf mpich2-1.0.8.tar.gz
```

3. Navigate to the mpich2-1.0.8 directory 
```
cd mpich2-1.0.8
```

4. Execute the configuration file. 
```
 ./configure
```

5. Make the binary file 
```
make;
make install;
```

6. Copy the content of mpich2-1.0.8 directory to `/cluster/mpich2`
```
 cp ./ /cluster/mpich2/ -r
```

7. Add the environment variables to `/cluster/mpiuser/.bash_profile` file
``` 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/cluster/mpich2/bin
LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/cluster/mpich2/lib

export PATH
export LD_LIBRARY_PATH
```

8. Now use the source command to apply the environment variables. 
```
source .bash_profile
```

9. Create a file in `/cluster/mpiuser/mpd.hosts` and add the hostname of each node.
``` 
node-1
node-2
master-node
```

10. Create a secret file named `.mpd.conf` in the `/cluster/mpiuser/` directory and add a secret word in it. Also, change the permission of the file to read-write by the user mpiuser only.
```
secretword=frost

#save file and change the permissions
chmod 600 .mpd.conf
```


