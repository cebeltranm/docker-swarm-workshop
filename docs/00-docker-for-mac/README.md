# Docker for MAC

### Version

Check you have the latest version of docker installed:
```
$ docker version
```

Check whether the DOCKER environment variables are set:
```
$ env | grep DOCKER
DOCKER_HOST=tcp://192.168.99.100:2376
DOCKER_MACHINE_NAME=default
DOCKER_TLS_VERIFY=1
DOCKER_CERT_PATH=/Users/victoriabialas/.docker/machine/machines/default
```

if you see environment variables remove it with:
```
$ unset $(env | grep DOCKER | sed 's/\(.*\)=.*/\1/')
```

Know the docker server running:
```
$ docker info
```
### Networking
#### Port Mapping
```
$ docker run -d -p 80:80 -v $(pwd)/src/welcome-site/:/usr/share/nginx/html --name webserver nginx:1.11.4-alpine
```

Check the container is running:
```
$ docker ps -a
```
and go to [http://localhost](http://localhost)

remove the containers
```
$ docker stop webserver && docker rm webserver
```

#### Bridge network

list the networks
```
$ docker network ls
```

create a new bridge network
```
$ docker network create --driver bridge workshop
```

list the networks to see if it was created.

run the backend
```
$ docker run -d --network workshop -v $(pwd)/src/welcome-site/:/usr/share/nginx/html --name backend nginx:1.11.4-alpine
```

inspect the network
```
$ docker network inspect workshop
```

ping the backend container from other container
```
$ docker run --rm -ti nginx:1.11.4-alpine ping backend
```

ping the backend container from other container connected to the same network
```
$ docker run --network workshop --rm -ti nginx:1.11.4-alpine ping backend
```

run a webserver as a proxy
```
$ docker run -d -p 80:80 --network workshop -v $(pwd)/src/proxy/:/etc/nginx/conf.d --name website nginx:1.11.4-alpine
```
and go to [http://localhost](http://localhost)

stop the backend container
```
$ docker stop backend && docker rm backend
```
and go to [http://localhost](http://localhost)

stop the website container
stop the backend container
```
$ docker stop website && docker rm website
```
