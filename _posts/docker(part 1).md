---
layout: post
title: "docker(part 1)"
subtitle: "images and container"
date: 2021-9-18
background: '/img/posts/06.jpg'
---
# how to get images:
[hub.docker.com](https://hub.docker.com/)  
you can search in docker hub,and get the images you want.
# how to run a container:
if you get a image,then you can start a container.
fisrt,if your os is windows,then open powershell and input this command to see all the images you got.

```
docker images
```
then you can start a container.

```
docker run --name mycontainer -d -p 8080:80 nginx:latest 
```
it can start a nginx container which name is mycontainer detached,meantime bind port 8080(localhost) to 80(nginx).  
after that,you can see the containers.

```
docker ps
```
what's more,you can use -a to see all the containers and -q list the containers' container id only.

```
docker ps -a
docker ps -q
docker ps -aq
```
# how to stop containers:
it's not difficult to know,the command to stop containers.you can stop containers by id or name.
```
docker stop mycontainer
docker stop [container id]
docker stop $(docker ps -aq)
```
# how to remove containers:
and if you want to remove containers,you can

```
docker rm [container id]
docker rm [container name]
docker rm $(docker ps -aq)
```