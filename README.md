# Private docker registry

[![Build](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)


## Description

Docker Registry is a server-side application and part of Docker’s platform-as-a-service product. It allows you to locally store all your Docker images into one centralized location. When you set up a private registry, you assign a server to communicate with Docker Hub over the internet. The role of the server is to pull and push images, store them locally, and share them among other Docker hosts.By running an externally-accessible registry, you can save valuable resources and speed up processes. The software lets you pull images without having to connect to the Docker Hub, saving up bandwidth and securing the system from potential online threats. Docker hosts can access the local repository over a secure connection and copy images from the local registry to build their own containers.

## Pre-Requests
- Need to install docker and docker-compose

### Docker installation 

```sh
yum install docker -y after docker installation, please start and enable it
```
### docker-compose installation

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
docker-compose version   
```
> You need to create a SSL certificate for the domain which you are using. Here am created a ssl certificate from https://www.sslforfree.com/ which is very simple using DNS validation.
> The created certs where placed on the /certs folder as below and created a HTTP authentication using httpd-tools

```
#auth~/ htpasswd -Bc registry.password username      ### This will create a authentication file under auth folder
```
```
~]# tree
.
├── auth
│   └── registry.password
├── certs
│   ├── server.crt
│   └── server.key
└── docker-compose.yml

2 directories, 4 files
```
```sh
version: '3'
services:
  registry:

    image: registry:2
    ports:
    - "5000:5000"

    environment:

      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/registry.password
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
      - REGISTRY_HTTP_ADDR=0.0.0.0:5000
      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt
      - REGISTRY_HTTP_TLS_KEY=/certs/server.key

    volumes:
      - data:/data
      - ./certs:/certs
      - ./auth:/auth

    networks:
      - registry_net

  frontend:
    environment:
     - ENV_DOCKER_REGISTRY_HOST=registry
     - ENV_DOCKER_REGISTRY_PORT=5000
     - ENV_DOCKER_REGISTRY_USE_SSL=1
     - ENV_USE_SSL="yes"
    volumes:
     - ./certs/server.crt:/etc/apache2/server.crt:ro
     - ./certs/server.key:/etc/apache2/server.key:ro
    ports:
     -  443:443
    image: konradkleine/docker-registry-frontend:v2
    networks:
     - registry_net

volumes:
  data:
networks:
  registry_net:
  ```
