# Docker Swarm First Steps

This project was created to help who wants to start using Docker Swarm.

<!-- ## Labs: 

- Testar acesso direto pelo IP dos workers
	Resultado: Acesso funciona sem precisar ir pelo master
- Testar acesso por IP de um worker que nao tem o container do serviço
	Resultado: Independente de qual worker for acessado, o acesso é redirecionado para o worker correto
- Deletar a interface web e reinstalar pra ver se perde conf
	Resultado: Nenhuma configuraçao do cluster de Swarm foi perdida
- Tentar expor novas portas após a criação do serviço
	Resultado: As novas portas são expostas sem interrupçao do serviço
- Queda do master
	Resultado: Cluster continua funcionando em caso de queda do nó master. Só não é possível fazer nenhuma alteração no cluster.
- Queda de container de um serviço qualquer
	Resultado: Automaticamente um novo container é iniciado para o serviço
- Testar backup
- Simular perda do nó master (todos) e testar o restore em um servidor master novo -->

## Some Commands

- `docker node ls`: Show nodes
- `docker service ls`: Show services
- `docker service rm SERVICE`: Remove some specific service
- `docker node rm NODE`: Delete node
- `docker swarm join-token worker`: Show command used to worker join in the cluster
- `docker swarm leave`: Leave the cluster
- `docker service logs -f SERVICE`: Show logs from some service
- `docker service inspect --pretty SERVICE`: Show information about some service
- `docker service update --image IMAGE:VERSION SERVICE`: Update image on service

## Installing Docker on CentOS 7

### Removing old packages 
```
yum remove docker \
           docker-common \
           docker-selinux \
           docker-engine
```

### Install required packages
```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

### Add repository
```
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

### Install docker comunity edition
```
yum install -y docker-ce
```

### Start and enable service on startup
```
systemctl start docker
systemctl enable docker
```

## Installing Docker on Ubuntu

### Update repository cache
```
apt-get update
```

### Install packages to allow apt to use a repository over HTTPS:
```
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

### Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### Set a stable repository
```
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

### Rebuild apt-get base
```
apt-get update
```

### Install docker-ce
```
apt-get install -y docker-ce
```

## Create Docker Swarm Cluster

#### Iniciar um cluster de Swarm (Executed on master only)
```
docker swarm init --advertise-addr MASTER_IP_ADDRESS
```

## Web Gui

### Option 1 - Portainer
```
docker service create --name SwarmWebgui --publish 9000:9000 --constraint 'node.role == manager' --mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock portainer/portainer -H unix:///var/run/docker.sock
```

### Option 2 - Rancher
```
docker service create --name SwarmWebgui --publish 8080:8080 --constraint 'node.role == manager' rancher/server:stable
```

### Option 3 - Shipyard
```
curl -s https://shipyard-project.com/deploy | bash -s
```

### Option 4 - Swarmpit
```
git clone https://github.com/swarmpit/swarmpit
docker stack deploy -c swarmpit/docker-compose.yml swarmpit
```

## Test of service

### Create service named Nginx (Just for testing)
```
docker service create --name Nginx \
--replicas 1 \
--publish 80:80 \
--update-delay 10s \
--update-parallelism 1 \
--constraint 'node.role != manager' \
nginx:latest
```

## Backup and Restore

### Backup

- Chose one of master node and stop docker service with comando: systemctl stop docker
- Backup swarm directory (/var/lib/docker/swarm)

### Restore

- Shut down Docker on the target host machine where the swarm will be restored.
- Remove the contents of the /var/lib/docker/swarm directory on the new swarm.
- Restore the /var/lib/docker/swarm directory with the contents of the backup.
- Start Docker on the new node. Unlock the swarm if necessary. 
- Re-initialize the swarm using the following command: `docker swarm init --force-new-cluster`
<!-- - Restart Docker service with command: `systemctl restart docker` -->

## Authors

* **Diogo Munhoz** - *Initial work* - [digmunhoz](https://github.com/digmunhoz)