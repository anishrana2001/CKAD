
### Task:   kubectl config use-context k8s-c1-s
### 1. A Dockerfile is created on /datadir/Dockerfile for you. You need to create built an image with the name ubuntu-apache and tag 3.0 from this Dockerfile. You may install and use the tool of your choice.
### 2. Using the tool of your choice export the built container image in OC-format and store it on /data1/ubuntu-apache-3.0.tar
### 3. Create a pod named apache-pod1 from the newly created image and bind apache port (80) with 34080 port number.

### Solution

### kubectl config use-context k8s-c1-s

### First, check the file.
```
cd /datadir/
```
###
```
cat Dockerfile 
# Usage: FROM [image_name]
FROM ubuntu:14.04
MAINTAINER Anish Rana
ENV TZ=Asia/Dubai
RUN ln -snf /usr/share/zoneinfo/ /etc/localtime && echo  > /etc/timezone
```

### We know that we can built the container's image from Dockerfile. 
```
docker build -t ubuntu-apache:3.0  .
```

### Post check !
```
docker image ls
```

### Create a pod named apache-pod1 from the newly created image and bind apache port (80) with 34080 port number.

```
docker run -d -it --name apache-pod1 -p 34080:80 ubuntu-apache:3.0
```

### Let's check the container.
```
docker ps
```

### We can also check the container details with the help of "inspect" command.
```
docker container inspect apache-pod1 | grep -i port
```
