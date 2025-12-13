Installation :-

Instance t3 large
Connect instance with git bash
ssh client
sudo apt-get update && sudo apt-get upgrade
Install docker - Official doccumentation.
kubectl
Terraform
Clone project : https://github.com/iam-veeramalla/ultimate-devops-project-demo
df -h: check file system details
sudo growpart /dev/xvda 1 : Alots the storage to a partition(default).
sudo resize2fs /dev/xvda1 : Confirm the resizing of the partition.
lsblk -> for disks partition
docker compose up -d... for starting the project

#### Docker Containerization

- Verify from the developer if there is any build/optimsed version to be done.
- If there is then first install the language.
- Test the build, if success only then make the dockerfile. It's the standard practice.

#### Requirements

- Programming language version
- Dependency file & command for dwnld the dependency.
- Command to create a build.
- ENV variables if any.
- PORT
- Entrypoint
  Note: In node specific apps it is mandatory to specify a user & give required permissions.

Dockerfile :-

- Refer Dockerfile in the services in src directory.
- Make a Dockerfile
- Make a build : docker build -t randeepgreyghost94/name:latest .
- Run the container.

Go-lang

## Stage-1

## Import docker image

FROM golang:1.22-alpine AS Builder

## Make a new directory

WORKDIR /usr/src/app

## Copy the code into new directory

## Copy the list of dependencies into new directory

COPY go.mod go.sum ./

## Install golang dependencies

RUN go mod download

## Copy all the files/code into new directory.

COPY . .

## Build

RUN go build -o product-catalog .

## Stage-2

## Import docker image

FROM alpine AS Release

## Make a new directory

WORKDIR /usr/src/app

COPY ./products/ ./products/

## Copy Stage-1 build code into new directory

COPY --from=Builder /usr/src/app/product-catalog ./

## Setup ENV variable

ENV PRODUCT_CATALOG_PORT=8100

ENTRYPOINT ["./product-catalog"]

####################################################################

java

# Import docker base image.

FROM eclipse-temurin:21-jdk AS builder

WORKDIR /usr/src/app

COPY gradlew gradlew.bat build.gradle settings.gradle / gradlew* build.gradle settings.gradle* .
COPY ./gradle ./gradle

RUN chmod +x ./gradlew
RUN ./gradlew
RUN ./gradlew downloadRepos

COPY . .
COPY ./pb ./proto
RUN chmod +x ./gradlew
RUN ./gradlew installDist -PprotoSourceDir=./proto

############################################################

FROM eclipse-temurin:21-jre AS release

WORKDIR /usr/src/app

COPY --from=builder /usr/src/app ./

ENV AD_PORT 2020

ENTRYPOINT []

############################################################

Node j.s

FROM node:24-alpine AS builder

WORKDIR /usr/src/app

COPY package.json package-lock.json ./

RUN npm ci

COPY . .

#############################

FROM node:24-alpine AS release

WORKDIR /usr/src/app

USER node

COPY --from=builder --chown=node:node /usr/src/app ./

COPY ./pb/demo.proto ./

ENV NODE_ENV=production

EXPOSE ${PAYMENT_PORT}

ENTRYPOINT ["npm","run","start"]

#### Deploying docker images on docker hub

- docker login docker.io
- docker push docker.io/randeepgreyghost94/container-name:version

#### Docker compose up

- Docker compose is similar to docker & developed by docker inc but the key difference is that docker compose is used to manage multi container application.
- It helps to manage container lifecycles & saves effort from running build & run commands multiple times.
- Only up command starts all the containers at once & stops all the services/containers with down command.
- We need to write the Docker files & a single docker-compose yaml file.
- The docker compose file will build & Run all the microservices or container.
- Ctrl + c will just stop the container unlike down cmd which will remove the containers.
- Docker compose is always a YAML file.
- Template :
  version: "3.9"

services:
app:
build:
context: .
dockerfile: Dockerfile
target: release
image: randeepgreyghost94/node-todo:v1
environment: - PORT=8000
restart: unless-stopped
networks: - web

nginx:
image: nginx:stable-alpine
depends_on: - app
ports: - "80:80"
volumes: - ./nginx/conf.d:/etc/nginx/conf.d:ro
networks: - web

networks:
web:
driver: bridge

#### Docker network

Bridge - It's a default network in docker where 2 containers can connect with each other which has different subnets.Without bridge a container cannot talk to the host. - We can create multiple custom bridge networks.
Host - If the subnet of your container and docker network is same then anyone can access the app & this is called host network.

#### nginx

# Section-1

-nginx is an open-source web server which is also used as Reverse-proxy, LB, HTTP-caching.

- Installation :- HTTP port 80 & HTTPS port 443 on t2 micro with nginx installation(cmds in repo).
- The configuration of nginx is in /etc/nginx.
- nginx.conf file has all the settings,logging,configurations etc.
- sites-available folder has configurations files of server blocks of static pages.
- Theses files cannot be active until linked to sites-enabled directory.
- sites-enabled contains symlinks to files in sites-available. Only configs here are actually loaded on nginx.

# Section-2

- nginx as web server to use static files/pages like HTML, JS, CSS, png etc is used to load over HTTP.
- var/www/html : It is a default root directory for static files.
- /etc/nginx/sites-available/default : It contains default config file pointing to the root.
- The static file in the root directory will be loaded on web browser only if it is linked/pointed to /sites-avail default directory.
- There can be multiple files in root dir & also multiple default links in sites-avail.
- sites-enabled/default is just a symbolic link for additional confirmation. The default in sites-enabled allows the content to be loaded on web server.
- Refer github repo for : Demo: Serve a Static Website Using NGINX

# Section-3

- Refer github repo.

# Section-4

- Load balancing is the process of distributing incoming network traffic across multiple backend servers.

Benefits:

Prevents server overload
Increases availability and fault tolerance
Enables horizontal scaling
NGINX supports multiple load balancing algorithms out of the box.

#### Fwd proxy & Rvse proxy

CLient - Fwd proxy
Server - Rvse proxy

#### Helm

- https://github.com/iam-veeramalla/helm-zero-to-hero/blob/main/01-what-is-helm.md

Yaml :- Refer file in Devops folder.

Ingress :- Kubernetes resource & controller for using LB of different enterprises.Refer veeramalla

SSL - security, Lecture with sinha

user www-data;
worker_processes auto;

events {
worker_connections 768;
}

http {
include /etc/nginx/mime.types;
server_tokens off;
sendfile on;
#server { # root /var/www/demo; # index randeep.html; # location = /about { # try_files /about.html =404; # }
#}

        server {
                location / {
                        proxy_pass http://localhost:3000;
                }
        }

}

user www-data;
worker_processes auto;

events {
worker_connections 768;
}

http {
server_tokens off;
sendfile on;
server {
root /var/www/demo;
index demo.html;
location = /about {
try_files /about.html =404;
}
}

        #server {
        #       location / {
        #               proxy_pass http://localhost:8000;
        #       }



    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/greyghost-todo.work.gd/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/greyghost-todo.work.gd/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

        server {
    if ($host = greyghost-todo.work.gd) {
        return 301 https://$host$request_uri;
    } # managed by Certbot




                listen 80;
                server_name greyghost-todo.work.gd www.greyghost-todo.work.gd;
    return 404; # managed by Certbot

}}

user www-data;
worker_processes auto;

events {
worker_connections 768;
}

http {
server_tokens off;
sendfile on;
server {
server_name greyghost-todo.work.gd www.greyghost-todo.work.gd;
root /var/www/demo;
index demo.html;

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/greyghost-todo.work.gd/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/greyghost-todo.work.gd/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

        server {
    if ($host = greyghost-todo.work.gd) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


                listen 80;
                server_name greyghost-todo.work.gd www.greyghost-todo.work.gd;
    return 404; # managed by Certbot

}}

user www-data;
worker_processes auto;

events {
worker_connections 768;
}

http {
server_tokens off;
sendfile on;
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
server {
root /var/www/demo;
index demo.html;
location = /about {
try_files /about.html =404;
}
#location / { # proxy_pass http://localhost:3000; # proxy_set_header Host $host; # proxy_set_header X-Real-IP $remote_addr; # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # proxy_set_header X-Forwarded-Proto $scheme;
#}

                server_name greyghost-back.work.gd www.greyghost-back.work.gd;

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/greyghost-back.work.gd/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/greyghost-back.work.gd/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

        server {
    if ($host = greyghost-back.work.gd) {
        return 301 https://$host$request_uri;
    } # managed by Certbot



                listen 80;
                server_name greyghost-back.work.gd www.greyghost-back.work.gd;
    return 404; # managed by Certbot

}}
