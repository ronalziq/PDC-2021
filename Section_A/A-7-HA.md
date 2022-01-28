# Step 1: Add IP address of both Machines in Serve Host File:

## Machine 1:

vim /etc/hosts

192.168.198.131/24 masternode

192.168.198.133/24 node1

![](RackMultipart20220126-4-13yq7dm_html_fa42065c06bb22b7.png)

![](RackMultipart20220126-4-13yq7dm_html_b72e5642aa487188.png)

## Machine 2:

vim /etc/hosts

192.168.198.131/24 masternode
`
192.168.198.133/24 node1

![](RackMultipart20220126-4-13yq7dm_html_7fc328e1988abbc3.png)

![](RackMultipart20220126-4-13yq7dm_html_8f4b4860e227477e.png)

# Step 2: Ping both machines from machine 1 to check connectivity:

## Ping Master Node Machine from Master Node Machine

ping masternode

![](RackMultipart20220126-4-13yq7dm_html_fbffd2a22c03399b.png)

## Ping Node 1 Machine from Master Node Machine

Ping node1

![](RackMultipart20220126-4-13yq7dm_html_399698102a044b65.png)

## Ping Node 1 from Node 1

ping node1

![](RackMultipart20220126-4-13yq7dm_html_45c193295f82456d.png)

## Ping Master Node Machine from Node 1

Ping masternode

![](RackMultipart20220126-4-13yq7dm_html_2ed40fbc76105526.png)

#

# Step 3: Flush iptables firewall on servers in order to configure servers without iptables firewall:

## Master Node Machine:

iptables –F

iptables -L

![](RackMultipart20220126-4-13yq7dm_html_bd070831db72d451.png)

iptables-save

![](RackMultipart20220126-4-13yq7dm_html_8b9377958df33656.png)

## Node 1 Machine

Iptables –F

Iptables -L

![](RackMultipart20220126-4-13yq7dm_html_d4145364ab818642.png)

iptables-save

![](RackMultipart20220126-4-13yq7dm_html_dd2a001d415595eb.png)

# Step 4 Configure NTP in Both Machines:

## Master Node Machine:

### Check NTP Status:

rpm –qa | grep ntp

![](RackMultipart20220126-4-13yq7dm_html_c4489476923baada.png)

###

### Configure as a Server:

vim /etc/ntp.conf

![](RackMultipart20220126-4-13yq7dm_html_92e8ef35a3e2b253.png)

server 0.asia.pool.ntp.org

server 1.asia.pool.ntp.org

server 2.asia.pool.ntp.org

server 3.asia.pool.ntp.org

server 127.127.1.0

![](RackMultipart20220126-4-13yq7dm_html_ac123d6d18a8773b.png)

### Restart NTP Server:

service ntpd restart

![](RackMultipart20220126-4-13yq7dm_html_e579b64284aaa08c.png)

##

## Node 1 Machine:

### Check NTP Status:

![](RackMultipart20220126-4-13yq7dm_html_9de5c73fc8aac63a.png)

### Configure as a Client:

vim /etc/ntp.conf

![](RackMultipart20220126-4-13yq7dm_html_69b2e3e4b60da55b.png)

server 192.168.198.131

![](RackMultipart20220126-4-13yq7dm_html_a35ee374487bc2b4.png)

###

### Restart NTP Server:

Service ntpd restart

![](RackMultipart20220126-4-13yq7dm_html_bba7330bf4b705a0.png)

### Sync Client Date and Time with Server (Master Node Machine):

ntpdate –u 192.168.198.131

![](RackMultipart20220126-4-13yq7dm_html_ea4681f84933e824.png)

# Check Server and Client Synchronization:

## Master Node Machine (Server):

ntpq –p -n

![](RackMultipart20220126-4-13yq7dm_html_883e79ed2dabb078.png)

timedatectl

![](RackMultipart20220126-4-13yq7dm_html_2fb4f46d0353ec62.png)

## Node 1 Machine (Client):

ntpq –p -n

![](RackMultipart20220126-4-13yq7dm_html_96c286dc510482f1.png)

timedatectl

![](RackMultipart20220126-4-13yq7dm_html_5a58c7929e921363.png)

# Step 5 Create Partitions on Both Machines For DRDB:

## Master Node Machine (Server):

### Create Partition:

fdisk /dev/sdb

![](RackMultipart20220126-4-13yq7dm_html_e9fc061e434f65b8.png)

### Change partition Type:

![](RackMultipart20220126-4-13yq7dm_html_32bf44621bd23114.png)

### Verify Partition:

fdisk -l

![](RackMultipart20220126-4-13yq7dm_html_93af8a084101e374.png)

## Node 1 Machine (Client):

### Create Partition:

fdisk /dev/sdb

![](RackMultipart20220126-4-13yq7dm_html_fb5c6020fe25b925.png)

### Change partition Type:

![](RackMultipart20220126-4-13yq7dm_html_3ad5ea973bf03368.png)

### Verify Partition:

fdisk -l

![](RackMultipart20220126-4-13yq7dm_html_aaec54c7616a62b8.png)

# Step 6 Create VG and LVM:

## Master Node Machine (Server):

vgcreate vgdrbd /dev/sdb1

lvcreate –n lvdrbd /dev/mapper/vgdrbd –L +5000M

![](RackMultipart20220126-4-13yq7dm_html_1b7c67d7eef09575.png)

## Node 1 Machine (Client):

vgcreate vgdrbd /dev/sdb1

lvcreate –n lvdrbd /dev/mapper/vgdrbd –L +5000M

##
 ![](RackMultipart20220126-4-13yq7dm_html_f62a3f4a507d4b76.png)

# Step 7 Create arp table validation:

## Master Node Machine (Server):

### Configure

vim /etc/sysctl.conf

![](RackMultipart20220126-4-13yq7dm_html_6f77993cc40206c6.png)

net.ipv4.conf.ens33.arp\_ignore = 1

net.ipv4.conf.all.arp\_announce = 2

net.ipv4.conf.ens33.arp\_announce = 2

![](RackMultipart20220126-4-13yq7dm_html_ea8fba2fabd13016.png)

###

Check Response:

sysctl -p

![](RackMultipart20220126-4-13yq7dm_html_403dbdbd2c21f7a9.png)

## Node 1 Machine (Client):

### Configure:

vim /etc/sysctl.conf

![](RackMultipart20220126-4-13yq7dm_html_9f10993ac072f4c5.png)

net.ipv4.conf.ens33.arp\_ignore = 1

net.ipv4.conf.all.arp\_announce = 2

net.ipv4.conf.ens33.arp\_announce = 2

![](RackMultipart20220126-4-13yq7dm_html_7fc88bb79492ad79.png)

Check Response

sysctl -p

![](RackMultipart20220126-4-13yq7dm_html_48d2b2204f7829a1.png)

# Step 8 Configure DRDB:

## Master Node Machine (Server):

###

### Add repl repository:

rpm -ivh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

![](RackMultipart20220126-4-13yq7dm_html_111976650cc83f77.png)

### Add GPG-Key to encrypt communication between nodes:

rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org

![](RackMultipart20220126-4-13yq7dm_html_9dd764e3260a23ec.png)

### Check drdb version supported by our kernel:

yum info \*drbd\* | grep Name

![](RackMultipart20220126-4-13yq7dm_html_fdea48003212f899.png)

### Install drdb:

yum –y install drbd84-utils kmod-drbd84

![](RackMultipart20220126-4-13yq7dm_html_a8c3763a53daecd6.png)

###

###

###

###

### Configure drbd file:

vim /etc/drbd.conf

![](RackMultipart20220126-4-13yq7dm_html_55ec6d7251f869c5.png)

vim /etc/drbd.conf

global {

usage-count yes;

}

common {

syncer { rate 10M;}

}

resource r0 {

protocol C;

handler{

pri-on-incon-degr &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

pri-lost-after-sb &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

local-io-error &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

outdate-peer &quot;/usr/lib/heartbeat/drbd-peer-outdater -t 5&quot;;

}

startup{

}

disk{

on-io-error detach;

}

net{

after-sb-0pri disconnect;

after-sb-1pri disconnect;

after-sb-2pri disconnect;

rr-conflict disconnect;

}

syncer{

rate 10M;

al-extents 257;

}

on masternode{

device /dev/drbd0;

disk /dev/vgdrbd/lvdrbd;

address 192.168.198.131:7788

meta-disk internal;

}

on node1{

device /dev/drbd0;

disk /dev/vgdrbd/lvdrbd;

address 192.168.198.131:7788

meta-disk internal;

}

}

![](RackMultipart20220126-4-13yq7dm_html_9c57d277b6cf1177.png)

![](RackMultipart20220126-4-13yq7dm_html_b0e9e07948b559a2.png)

###

### Add drbd module in kernel:

modprobe drbd

![](RackMultipart20220126-4-13yq7dm_html_e73af9472c160dd7.png)

### To make the modules be loaded during each boot:

echo &quot;modprobe drbd&quot; \&gt;\&gt; /etc/rc.local

![](RackMultipart20220126-4-13yq7dm_html_2a5f8fbf87aff4db.png)

### Allow metadata storage on each step:

drbdadm create-mod r0

![](RackMultipart20220126-4-13yq7dm_html_ea64b47c29811bc6.png)

### Fix Drbdsetup and Drbdmeta:

groupadd haclient

chgrp haclient /lib/drbd/drbdsetup-84

chmod o-x /lib/drbd/drbdsetup-84

chmod u+s /lib/drbd/drbdsetup-84

chgrp haclient /usr/sbin/drbmeta

chmod o-x /usr/sbin/drbmeta

chmod u+s /usr/sbin/drbmeta

![](RackMultipart20220126-4-13yq7dm_html_6def66816cd3a1e6.png)

### Attach the drbd0 device to sdb1 disk:

drbdadm up r0

![](RackMultipart20220126-4-13yq7dm_html_8908c55ea13dbd2c.png)

### Assign Ports in Firewall:

firewall-cmd --zone=internal --add-port=7788-7799/tcp --permanent

firewall-cmd --zone=internal --add-port=7788-7799/udp --permanent

firewall-cmd --reload

![](RackMultipart20220126-4-13yq7dm_html_f3a312be72eaaae1.png)

### Start and Enable DRBD:

![](RackMultipart20220126-4-13yq7dm_html_9ba1c11810167621.png)

### specify masternode as primary node:

drbdadm primary r0 --force

![](RackMultipart20220126-4-13yq7dm_html_d49ca04ac26fa80f.png)

### Create a filesystem on the device:

mkfs –t ext4 /dev/drbd0

![](RackMultipart20220126-4-13yq7dm_html_459d9bbcb0c4f067.png)

### Check the status of drbd process:

cat /proc/drbd

![](RackMultipart20220126-4-13yq7dm_html_f88d11489dcf9351.png)

### Assign File System to DRBD deviice

mkfs –t ext3 /dev/drbd0

![](RackMultipart20220126-4-13yq7dm_html_7d8932f1e27b5a1.png)

### Create directory:

mkdir /data/

![](RackMultipart20220126-4-13yq7dm_html_b8707ab515cc8740.png)

### Mount Directory:

mount /dev/drbd0 /data/

![](RackMultipart20220126-4-13yq7dm_html_61b2a433ef5383cb.png)

### Check data directory:

cd /data/

![](RackMultipart20220126-4-13yq7dm_html_d1badcaf1ca88d2d.png)

### Check drbd0 drive is mounted:

df -h

### ![](RackMultipart20220126-4-13yq7dm_html_a658f39ee2805228.png)

## Node 1 Machine (Client):

### Add repl repository:

rpm -ivh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm

![](RackMultipart20220126-4-13yq7dm_html_8ccd8d1894c57620.png)

### Add GPG-Key to encrypt communication between nodes:

rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org

![](RackMultipart20220126-4-13yq7dm_html_f8b723a03d65edbe.png)

### Check drdb version supported by our kernel:

yum info \*drbd\* | grep Name

![](RackMultipart20220126-4-13yq7dm_html_10b6c33845c38304.png)

### Install drdb:

yum -y install drbd-utils kmod-drbd84

![](RackMultipart20220126-4-13yq7dm_html_d0605d455a4bc91.png)

### Configure drbd file

vim /etc/drbd.conf

![](RackMultipart20220126-4-13yq7dm_html_bd07d4c0136cdbd8.png)

global {

usage-count yes;

}

common {

syncer { rate 10M;}

}

resource r0 {

protocol C;

handler{

pri-on-incon-degr &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

pri-lost-after-sb &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

local-io-error &quot;echo o \&gt; /proc/sysrq-trigger; halt -f &quot;;

outdate-peer &quot;/usr/lib/heartbeat/drbd-peer-outdater -t 5&quot;;

}

startup{

}

disk{

on-io-error detach;

}

net{

after-sb-0pri disconnect;

after-sb-1pri disconnect;

after-sb-2pri disconnect;

rr-conflict disconnect;

}

syncer{

rate 10M;

al-extents 257;

}

on masternode{

device /dev/drbd0;

disk /dev/vgdrbd/lvdrbd;

address 192.168.198.131:7788

meta-disk internal;

}

on node1{

device /dev/drbd0;

disk /dev/vgdrbd/lvdrbd;

address 192.168.198.131:7788

meta-disk internal;

}

}

![](RackMultipart20220126-4-13yq7dm_html_9c57d277b6cf1177.png)

![](RackMultipart20220126-4-13yq7dm_html_b0e9e07948b559a2.png)

### Add drbd module in kernel:

### modprobe drbd

### lsmod | grep –i drbd
 ![](RackMultipart20220126-4-13yq7dm_html_d6408d8c2b9316b9.png)

### To make the modules be loaded during each boot:

echo &quot;modprobe drbd&quot; \&gt;\&gt; /etc/rc.local

![](RackMultipart20220126-4-13yq7dm_html_e40308d67a0455e3.png)

### Allow meta data storage on each nodes:

drbdadm create-md r0

![](RackMultipart20220126-4-13yq7dm_html_79b47abfb4615243.png)

### Fix Drbdsetup and Drbdmeta:

groupadd haclient

chgrp haclient /lib/drbd/drbdsetup-84

chmod o-x /lib/drbd/drbdsetup-84

chmod u+s /lib/drbd/drbdsetup-84

chgrp haclient /usr/sbin/drbmeta

chmod o-x /usr/sbin/drbmeta

chmod u+s /usr/sbin/drbmeta

![](RackMultipart20220126-4-13yq7dm_html_7a8776561d321696.png)

### Attach the drbd0 device to sdb1 disk:

Drbdadm up r0

![](RackMultipart20220126-4-13yq7dm_html_fa64ad49de7d9ce8.png)

###

### Assign Ports in Firewall:

firewall-cmd --zone=internal --add-port=7788-7799/tcp --permanent

firewall-cmd --zone=internal --add-port=7788-7799/udp --permanent

firewall-cmd --reload

![](RackMultipart20220126-4-13yq7dm_html_2038e740d7eb3c48.png)

### Start and Enable DRBD:

systemctl start drbd

### ![](RackMultipart20220126-4-13yq7dm_html_41a15e09ebc2364a.png)

### Check the status of drbd process

cat /proc/drbd

### ![](RackMultipart20220126-4-13yq7dm_html_2cf2d8c1ca1d0fcb.png)

### Create Directory

mkdir /data/

![](RackMultipart20220126-4-13yq7dm_html_8d255c76a14915e3.png)

##