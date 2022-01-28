## HPC Cluster

An HPC cluster consists of hundreds or thousands of compute servers that are networked together. Each server is called a node. The nodes in each cluster work in parallel with each other, boosting processing speed to deliver high performance computing.


## Project

# Set up  nodes

In this step, install the cluster tools, configure and enable the MUNGE service, and enable the Slurm controller service.

Install Clear Cent OS  on the controller node, add a user with adminstrator privilege, and set its hostname to hpc-controller.

Boot it up and log in.

Change hotname using in all three nodes

```bash
vi /etc/sysconfig/hostname
```
Change ip address in all nodes

```bash
vi /etc/sysconfig/netwok-scripts
```
Change hots using in all three nodes

```bash
vi /etc/sysconfig/hosts
```
# Set up password-less SSH access and pdsh on all nodes¶
To efficiently manage a cluster, it is useful to have a tool that allows issuing the same command to multiple nodes at once. And that tool is pdsh, which is included with the cluster-tools bundle. pdsh is built with Slurm support, so it can access hosts as defined in the Slurm partitions. pdsh relies on password-less SSH access in order for it to work properly. There are two ways to set up pasword-less SSH authentication: key-based or host-based. In this case, the latter approach will be used. The controller authenticates a user and all worker nodes will trust that authentication and not ask the user to enter a password again.

Configure the controller node.

Log into the controller node.

Configure the SSH service for host-based authentication.
```bash
Create	a	SSH	keys	folder.
mkdir	 ~/.ssh
```
generate	ssh	keys
```bash
ssh-keygen	 -t	rsa	 -b	4096	 -C	your_email@example.com
```
You can press Enter to leave the next three prompts as default
O
pen	the	ssh	folder
```bash
cd ~/.ssh
```
copy	the	public	key,	id_rsa.pub,	to	authorized_keys	to	enable	this	key	for	access	to	machine	#1
```bash
cp id_rsa.pub	authorized_keys
```

Now,	we	should	send	the	private	key,	id_rsa,	and	public	key,	id_rsa.pub,	from	machine	#1	to	machine	
#2.	We	use	a	command	called	scp	for	copying	files	over	machines.
```bash
scp	 ~/.ssh/id_rsa	~/.ssh/id_rsa.pub	root@10.0.1.3:
```
On	machine	#2,	we	have	received	the	private	key	and	public	key.	We	need	to	make	the	~/.ssh	directory	
on	machine	#2.
```bash
mkdir	 ~/.ssh
```
Now,	we	copy	the	id_rsa	and	id_rsa.pub	to	the	~/.ssh	folder.
```bash
cp	id_rsa	id_rsa.pub	 ~/.ssh
```
We want to copy id_rsa.pub to the authorized_keys to allow machine #1 to be able to SSH
to machine #2 without a password.
```bash 
cd	~/.ssh
cp	id_rsa.pub	authorized_keys
```
We should be able to ssh from machine #1 to machine #3 without a password and vice
versa.

```bash
On	machine	#1:	ssh	root@master
On	machine	#2:	ssh	root@node1
On	machine	#3:	ssh	root@node2
```
# Create slurm.conf configuration file

On the controller, create a new slurm.conf configuration file that contains general settings, each node’s hardware resource information, grouping of nodes into different partitions, and scheduling settings for each partition. This file will be copied to all worker nodes in the cluster

Create a base slurm.conf configuration file.
 ```bash
sudo mkdir -p /etc/slurm
sudo cp -v /usr/share/defaults/slurm/slurm.conf /etc/slurm

 ```

 Add the controller information.

sudoedit the slurm.conf file. Set the ControlMachine value to the controller’s resolvable hostname.

```bash
ControlMachine=hpc-controller
```
Add the worker nodes information.

Create a file containing a list of the worker node

```bash
cat > worker-nodes-list << EOF
```

Using pdsh, get the hardware configuration of each node.

```bash
pdsh -w ^worker-nodes-list slurmd -C
```

udoedit the slurm.conf file. Append each worker node information, but without the UpTime, under the COMPUTE NODES section.

Create partitions.

A Slurm partition is basically the grouping of worker nodes. Give each partition a name and decide which worker node(s) belong to it.
```bash 
PartitionName=workers Nodes=hpc-worker1, hpc-worker2, hpc-worker3 Default=YES MaxTime=INFINITE State=UP
PartitionName=debug Nodes=hpc-worker1, hpc-worker3 MaxTime=INFINITE State=UP
```

Save and exit.

Set the ownership of the slurm.conf file to slurm.
```bash
sudo chown slurm: /etc/slurm/slurm.conf
```

On the controller node, restart the Slurm controller service.
```bash
sudo systemctl restart slurmctld
```

Verify the Slurm controller service restarted without any issues before proceeding.
```bash
sudo systemctl status slurmctld
```
# Copy MUNGE key and slurm.conf to all worker nodes

On the controller node, using pdsh, in conjunction with the list of defined nodes in the slurm.conf, copy it and the MUNGE key to all worker nodes.

On the controller node, copy the MUNGE key to all worker nodes and start the MUNGE service.

Create the /etc/munge/ directory on each node.
```bash
sudo pdsh -P workers mkdir /etc/munge
```
Copy the MUNGE key over.
```bash
sudo pdcp -P workers /etc/munge/munge.key /etc/munge
```
Set the ownership of the munge.key file to munge.
```bash
sudo pdsh -P workers chown munge: /etc/munge/munge.key
```
Start the MUNGE service and set it to start automatically on boot.
```bash
sudo pdsh -P workers systemctl enable munge --now
```
Verify the MUNGE service is running.
```bash
sudo pdsh -P workers "systemctl status munge | grep Active"
```
On the controller node, copy the slurm.conf file to all worker nodes and start the slurmd service on them.

Create the /etc/slurm/ directory on each worker node.
```bash
sudo pdsh -P workers mkdir /etc/slurm
```
Copy the slurm.conf file over.
```bash
sudo pdcp -P workers /etc/slurm/slurm.conf /etc/slurm
```
Set the ownership of the slurm.conf file to slurm.
```bash
sudo pdsh -P workers chown slurm: /etc/slurm/slurm.conf
```
Start the Slurm service and set it automatically start on boot.
```bash
sudo pdsh -P workers systemctl enable slurmd --now
```
Verify the slurmd service is running.
```bash
sudo pdsh -P workers systemctl status slurmd | grep Active
```
# Verify controller can run jobs on all nodes¶
Check the state of the worker nodes.
```bash
sinfo
```
And finally, verify Slurm can run jobs on all 4 worker nodes by issuing a simple hostname command.
```bash
srun -N4 -p workers hostname
```
