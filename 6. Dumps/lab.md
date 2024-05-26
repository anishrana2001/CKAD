![image](https://github.com/anishrana2001/CKAD/assets/93471182/f3ae2182-da1d-4429-a8ce-dee349ae967c)### Create a Lab.
```
mkdir /datadir/
mkdir /data1/
cat <<EOF>> /datadir/Dockerfile
# Usage: FROM [image_name]
FROM ubuntu:14.04
MAINTAINER Anish Rana
ENV TZ=Asia/Dubai
RUN ln -snf /usr/share/zoneinfo/ /etc/localtime && echo  > /etc/timezone
EOF


kubectl create namespace ckad0021
kubectl create namespace ns-quota1
kubectl -n ns-quota1 create deployment resource-deploy --image=nginx
cat << EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: security-deploy
  namespace: ckad0021
  labels:
    app: security-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: security-deploy
  strategy: {}
  template:
    metadata:
      labels:
        app: security-deploy
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
EOF

cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mem-cpu-demo
  namespace: ns-quota1
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
EOF

```



### Task:   kubectl config use-context k8s-c1-s
### 1. A Dockerfile is created on /datadir/Dockerfile for you. You need to create built an image with the name ubuntu-apache and tag 3.0 from this Dockerfile. You may install and use the tool of your choice.
### 2. Using the tool of your choice export the built container image in OC-format and store it on /data1/ubuntu-apache-3.0.tar
### 3. Create a container named apache-pod1 from the newly created image and bind apache port (80) with 34080 port number.

### Solution

### kubectl config use-context k8s-c1-s

### First, check the file. For task 1.
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

### For task 2, examiner asked us to create a .tar file from this image.

```
sudo docker save ubuntu-apache:3.0 > /data1/ubuntu-apache-3.0.tar
```

### Post check for task 2.
```
ls -l /data1/ubuntu-apache-3.0.tar
```

### Task 3, Create a container named apache-pod1 from the newly created image and bind apache port (80) with 34080 port number.

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




### Question 2: 

### Task:  kubectl config use-context k8s-c1-s

### There is one deployment "resource-deploy" is running under namespace "ns-quota1". You need to set resource request (memory) half of the  max memory assigned to the namespace.


### Solution
### Use the right Context 
```
kubectl config use-context k8s-c1-s
```

### We can also check this ResourceQuota by executing below commands.

```
kubectl -n ns-quota1 describe resourcequota mem-cpu-demo
```

### Pre-checks: First check the deployment details.
```
kubectl -n ns-quota1 get deployment resource-deploy
```

### Validate, if memory quota is already set or not?

```
kubectl -n ns-quota1 get deployment resource-deploy -o yaml | grep -i mem
```

```
kubectl -n ns-quota1 edit deployment resource-deploy
```
-------------
In the container section
    resources:
      limits:
        memory: "1Gi"
----------------

### Post checks: Validate the memory quota.
```
kubectl -n ns-quota1 get deployment resource-deploy -o yaml | grep -i mem
```

### Check if our deployment is still running ?

```
kubectl -n ns-quota1 get deployments.apps
```




### Question 3: 

### Task:  kubectl config use-context k8s-c1-s

### There is one deployment "security-deploy" is running under namespace "ckad0021". You need to set securityContext user with 1000 forbide allowPrivilleged escalation.

### Solution


### use the correct context:
```
kubectl config use-context k8s-c1-s
```

### Precheck- Check the deployment in the given namespace.
```
kubectl -n ckad0021 get deployments.apps security-deploy 
```

### Validate the securityContext
```
kubectl -n ckad0021 get deployments.apps -o yaml | grep -A 1 securityContext
```
### we can directly edit this deployment.

```
kubectl -n ckad0021 edit deployments.apps security-deploy
```
-------------
      securityContext:
        runAsUser: 1000

        securityContext:
          allowPrivilegeEscalation: false
----------------

### PostChecks- Check the deployment's are still running?
```
kubectl -n ckad0021 get deployments.apps security-deploy 
```
### Post check 2 - Validate the securityContext
```
kubectl -n ckad0021 get deployments.apps -o yaml | grep -A 1 securityContext
```

### FYI that, if we login into the pod and execute "ps" command, we will observe 1000 PID only.
```
kubectl -n ckad0021  exec -it pods/security-deploy-5dfcfcf88f-nrtc2 -- /bin/sh
```
```
ps
```
```
touch file.txt
```


Clear the lab

```
kubectl -n ckad0021 delete deployments.apps/security-deploy 
kubectl delete namespaces/ckad0021
kubectl -n ns-quota1 delete deployment resource-deploy
kubectl -n ns-quota1 delete resourcequotas mem-cpu-demo
kubectl delete namespace ns-quota1
```





