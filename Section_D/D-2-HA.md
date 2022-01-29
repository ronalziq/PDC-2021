
# HA Cluster using Docker Swarm

High Availability cluster means the server is up and running all the time. To configure this cluster you need more than 1 pyhsical or virtual machine with docker installed and running which are called docker machines. Docker Swarm is an orchestration management tool that runs on Docker applications. It helps end-users in creating and deploying a cluster of Docker nodes. Each node of a Docker Swarm is a Docker daemon, and all Docker daemons interact using the Docker API. Docker Swarm consists of one manager node and multiple worker nodes. 

## Configuration

Once the docker machines are setup you need to chose which of these machine is going to be the manager node. 

```bash
 docker swarm init --advertise-addr 192.168.152.12
```
This command will create a cluster and generate a token which can then be used for adding worker nodes.
```bash
   docker swarm join --token SWMTKN-1-0t9ookqh94dxfvd1ccag79mt2wb6jmvv0mq00fz8vy0f3x1kcq-4hs5ivu27aym1l432rnzrpfy1 192.168.152.12:2377
```
Then you need to access the worker nodes and run the above command for them to join the cluster. You can run the following command to see machines which are available in the cluster(you can only run this command on the manager node).
```
docker node ls
```
In my case there is one manager node and two worker nodes.
```
node 1-manager
node 2-worker
node 3-worker
```
Then you need to deploy a docker stack, you will need a 'docker-compose.yml' file which will include the version, image and port.
docker-compose.yml:
```
version='3'
services: 
    web:
        image: nginx
        ports:"8080:80"
```
After, run the command
```
docker stack deploy --compose-file docker-compose.yml Mynginx
```
This command can only be executed on the manager node. It will deploy an image of 'nginx' on the port '8080', which is the port for 'http'.
You can access the webpage using any node's ip address and the port 8080.
```
192.168.152.12:8080
```
This can be accessed through a web browser or the bash terminal.
=======
# HA Cluster using Docker Swarm

High Availability cluster means the server is up and running all the time. To configure this cluster you need more than 1 pyhsical or virtual machine with docker installed and running which are called docker machines. Docker Swarm is an orchestration management tool that runs on Docker applications. It helps end-users in creating and deploying a cluster of Docker nodes. Each node of a Docker Swarm is a Docker daemon, and all Docker daemons interact using the Docker API. Docker Swarm consists of one manager node and multiple worker nodes. 

## Configuration

Once the docker machines are setup you need to chose which of these machine is going to be the manager node. 

```bash
 docker swarm init --advertise-addr 192.168.152.12
```
This command will create a cluster and generate a token which can then be used for adding worker nodes.
```bash
   docker swarm join --token SWMTKN-1-0t9ookqh94dxfvd1ccag79mt2wb6jmvv0mq00fz8vy0f3x1kcq-4hs5ivu27aym1l432rnzrpfy1 192.168.152.12:2377
```
Then you need to access the worker nodes and run the above command for them to join the cluster. You can run the following command to see machines which are available in the cluster(you can only run this command on the manager node).
```
docker node ls
```
In my case there is one manager node and two worker nodes.
```
node 1-manager
node 2-worker
node 3-worker
```
Then you need to deploy a docker stack, you will need a 'docker-compose.yml' file which will include the version, image and port.
docker-compose.yml:
```
version='3'
services: 
    web:
        image: nginx
        ports:"8080:80"
```
After, run the command
```
docker stack deploy --compose-file docker-compose.yml Mynginx
```
This command can only be executed on the manager node. It will deploy an image of 'nginx' on the port '8080', which is the port for 'http'.
You can access the webpage using any node's ip address and the port 8080.
```
192.168.152.12:8080
```
This can be accessed through a web browser or the bash terminal.

