# -docker swarm mode

### Create the servers

using docker machine we can create 3 servers (1 managers, 2 workers)
```
docker-machine create -d virtualbox manager-1
docker-machine create -d virtualbox worker-1
docker-machine create -d virtualbox worker-2
```

see the machines created
```
docker-machine ls
```

check the version of docker installed
```
eval "$(docker-machine env manager-1)"
docker info
```

### Install the Docker Swarm Visualizer

to see how the dockers and machine run in swarm we are going to use the [Docker Swarm Visualizer](https://github.com/ManoMarks/docker-swarm-visualizer).

```
eval "$(docker-machine env manager-1)"
docker run -d -p 8080:8080 -e HOST=$(docker-machine ip manager-1) -v /var/run/docker.sock:/var/run/docker.sock --name visualizer manomarks/visualizer
open http://$(docker-machine ip manager-1):8080
```

### Init the swarm

```
eval "$(docker-machine env manager-1)"
docker swarm init --advertise-addr $(docker-machine ip manager-1)
```
The command show the instructions to join more managers and workers to the swarm

add the other workers using the correct token
```
eval "$(docker-machine env worker-1)"
docker swarm join --token XXXXX
eval "$(docker-machine env worker-2)"
```
see the swarm activated in the manager-1
```
eval "$(docker-machine env manager-1)"
docker info
```
and list the nodes
```
docker node ls
```

### Deploy a published service
```
eval "$(docker-machine env manager-1)"
docker service create --name webserver --publish 80:80 nginx:1.11.4-alpine
```
go to the page to see the site:
```
open http://$(docker-machine ip worker-2)
```

list the services created
```
docker service ls
```

list the task
```
docker service ps webserver
```

connect to the server running webserver and stop the container, for example if the last command shows:
```
ID                         NAME         IMAGE                NODE       DESIRED STATE  CURRENT STATE           ERROR
d6sav28z3ja6s8l3prd2m6woh  webserver.1  nginx:1.11.4-alpine  worker-1 Running        Running 1 minutes ago
```
you should connect to the worker-1
```
eval "$(docker-machine env worker-1)"
docker stop webserver.1.d6sav28z3ja6s8l3prd2m6woh
```

wait for a moment and list again the tasks

#### scale the service
```
eval "$(docker-machine env manager-1)"
docker service scale webserver=4
```
list again the tasks

#### update the service
```
docker service update --image httpd:2-alpine  --update-delay 60s --update-parallelism 2 webserver
```
list again the tasks

#### destroy the service
```
docker service rm webserver
```

### Overlay Networking

create a new overlay network
```
$ docker network create --driver overlay workshop
```

create a new service connected to the network
```
docker service create --name webserver --replicas 3 --network workshop nginx:1.11.4-alpine
```

inspect the service
```
docker service inspect --format='{{json .Endpoint.VirtualIPs}}'  webserver
```

create a docker to execute commands in all the nodes:
```
docker service create --name commands --mode global --network workshop alpine sleep 3000
```

see the id of the container
```
docker service ps commands
```
and connect to it using the id, for example:
```
docker exec -ti commands.0.3o3d6r0977py9rhq2uji0iybz /bin/sh
```

run a nslookup to see where is the LB server
```
# nslookup webserver
```

run a nslookup to see where are the webservers
```
# nslookup tasks.webserver
```

### Manage nodes

list the nodes:
```
docker node ls
```

inspect a node
```
docker node inspect worker-1 --pretty
```

drain a node
```
docker node update --availability drain worker-2
```

re-active the node
```
docker node update --availability active worker-2
```

promote a node
```
docker node promote worker-2
```

stop the manager node, connect to the worker-2 and list the nodes
```
docker-machine stop manager-1
eval "$(docker-machine env worker-2)"
docker node ls
```

start the manager node, list the nodes
```
docker-machine start manager-1
docker node ls
```

promote the other worker, stop the new leader and list the nodes
```
docker node promote worker-2
docker-machine start worker-2
eval "$(docker-machine env worker-1)"
docker node ls
```

### remove the docker-machines
```
docker-machine rm worker-2
docker-machine rm worker-1
docker-machine rm manager-1
```
