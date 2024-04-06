



#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes
### Create a new deployment for running NGINX with the following parameters. 
#### - Run the deploymnet in the kdp0024 namespace. The namespace has alrady been created. 
#### - Name the deployment mydeploy and configure with 3 replicas
#### - Configure the pod with a container image of nginx:1.24.0
#### - Set an environment variable of NGINX_Port=8080 and also expose that port for the container above

### Solution: 

### Use the correct context:
```
[root@master1 ~]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

[root@master1 ~]# kubectl -n kdp0024  create deployment mydeploy  --replicas=3 --image=nginx:1.24.0 --port=8080  -o yaml --dry-run=client > mydeploy.yaml

[root@master1 ~]# vi mydeploy.yaml

[root@master1 ~]# cat mydeploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: mydeploy
  name: mydeploy
  namespace: kdp0024
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mydeploy
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: mydeploy
    spec:
      containers:
      - image: nginx:1.24.0
        name: nginx
        ports:
        - containerPort: 8080
        resources: {}
        env:
          - name: NGINX_Port
            value: "8080"
status: {}

[root@master1 ~]# kubectl apply -f mydeploy.yaml 
deployment.apps/mydeploy created

[root@master1 ~]# kubectl get deployments.apps -n kdp0024 
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
mydeploy   3/3     3            3           22s

[root@master1 ~]# kubectl -n kdp0024 get pods
NAME                       READY   STATUS    RESTARTS   AGE
mydeploy-d44f485d8-948v2   1/1     Running   0          58s
mydeploy-d44f485d8-dnmtj   1/1     Running   0          58s
mydeploy-d44f485d8-mvp4l   1/1     Running   0          58s

[root@master1 ~]# kubectl -n kdp0024 exec -it mydeploy-d44f485d8-948v2 -- printenv | grep NGINX_Port
NGINX_Port=8080
[root@master1 ~]# 
```











# Question: 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes
### As a kubernetes application developer you will often observe that sometime you need to update the running application. 
### -  Update the web deployment in the kdpd0023 namespace with a maxSurge of 10% and a maxUnavailable of 5%
### -  Perform a rolling update of the web deployment changing the nginx:1.24.0 image version to 1.24.1
### -  Perform the rollback the web deployment to the previous version 

### Solution: 

### Use the correct context:
```
[root@master1 ~]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

[root@master1 ~]# kubectl -n kdpd0023 create deployment web --image=nginx:1.24.0
deployment.apps/web created
[root@master1 ~]# kubectl edit deployment web -n kdp
kdp0024   kdpd0023  
[root@master1 ~]# kubectl edit deployment web -n kdpd0023 
Edit cancelled, no changes made.
[root@master1 ~]# 
[root@master1 ~]# 
[root@master1 ~]# 
[root@master1 ~]# kubectl -n kdpd0023 get deployments/web 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    1/1     1            1           93s
[root@master1 ~]# kubectl -n kdpd0023 get deployments/web -o yaml | grep max
      maxSurge: 25%
      maxUnavailable: 25%
[root@master1 ~]# kubectl -n kdpd0023 edit deployment web 
deployment.apps/web edited
[root@master1 ~]# kubectl -n kdpd0023 get deployments/web -o yaml | grep max
      maxSurge: 10%
      maxUnavailable: 5%
[root@master1 ~]# kubectl -n kdpd0023 get deployments/web -o yaml | grep image
      - image: nginx:1.24.0
        imagePullPolicy: IfNotPresent

[root@master1 ~]# kubectl -n kdpd0023 set image deployment web nginx=nginx:1.24.1
deployment.apps/web image updated
[root@master1 ~]# 

[root@master1 ~]# kubectl -n kdpd0023 get deployments/web -o yaml | grep image
      - image: nginx:1.24.1
        imagePullPolicy: IfNotPresent

[root@master1 ~]# kubectl -n kdpd0023 rollout history deployment web 
deployment.apps/web 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@master1 ~]# kubectl -n kdpd0023 rollout undo deployment web 
deployment.apps/web rolled back
[root@master1 ~]# kubectl -n kdpd0023 rollout history deployment web 
deployment.apps/web 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```

#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes

#### Developer occasionally need to submit pods that run periodically.
#### - Create a mainfest file /var/data/periodic.yaml in a YAML format.
#### - Yaml that runs the shell command "uname" in a single busybox container. 
#### - The command should run every minute and must complete within 28 seconds or be terminated by kubernetes. 
#### - The CronJob name and container name should both be hellocron
#### - Create the cronjob from the above manifest file.


#### If you are not aware about the crontjob systax, then you can use the help command.
```
[root@master1 ~]# kubectl create cronjob  -h
Create a cron job with the specified name.

Aliases:
cronjob, cj

Examples:
  # Create a cron job
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"
  
  # Create a cron job with a command
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *" -- date

Options:
    --allow-missing-template-keys=true:
	If true, ignore any errors in templates when a field or map key is missing in the template. Only applies to
	golang and jsonpath output formats.

    --dry-run='none':
	Must be "none", "server", or "client". If client strategy, only print the object that would be sent, without
	sending it. If server strategy, submit server-side request without persisting the resource.


[root@master1 ~]# kubectl create cronjob hellocron --image=busybox --schedule="*/1 * * * *" 
cronjob.batch/hellocron created

[root@master1 ~]# kubectl get cronjobs.batch 
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hellocron   */1 * * * *   False     0        <none>          8s
[root@master1 ~]# 
```

#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes

#### A deployment is failing on the cluster due an incorrect image being specifed. You need to locate the deployment and fix the issue.

### Solution: 

### Use the correct context:
```
[root@master1 ~]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

[root@master1 data]# kubectl get  deployment -A
NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       failed-deploy             0/1     1            0           2m8s
kdp0024       mydeploy                  3/3     3            3           163m
kdpd0023      web                       1/1     1            1           148m
kube-system   calico-kube-controllers   1/1     1            1           34d
kube-system   coredns                   2/2     2            2           34d
kube-system   metrics-server            2/2     2            2           34d
[root@master1 data]# 

[root@master1 data]# kubectl get deployments.apps failed-deploy -o yaml | grep -i image
      - image: ngin
        imagePullPolicy: Always

[root@master1 data]# kubectl edit deployments.apps failed-deploy 
deployment.apps/failed-deploy edited

[root@master1 data]# kubectl get  deployment -A
NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       failed-deploy             1/1     1            1           5m43s
kdp0024       mydeploy                  3/3     3            3           167m
kdpd0023      web                       1/1     1            1           152m
kube-system   calico-kube-controllers   1/1     1            1           34d
kube-system   coredns                   2/2     2            2           34d
kube-system   metrics-server            2/2     2            2           34d
[root@master1 data]# 

```



#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes

### A application is failling due to livenessprob. If can be run on any of the below namespaces.
#### - qa
#### - lab
#### - prod
#### - dev

### Task need to be performed:
### Identify the broken pod and write its name and namespace to /var/data/broken.txt in the format <namespace>/<pod>
### Copy the events into the file /var/data/error.txt. Use "-o wide" output specifier with your command.
### Fix the issue.

### Solution: 

### Use the correct context:
```
[root@master1 ~]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

[root@master1 data]# kubectl get po -A | egrep "qa|prod|dev|lab"
dev           liveness-http                                 0/1     CrashLoopBackOff   34 (2m43s ago)   81m

[root@master1 data]# echo "dev/liveness-http" > /var/data/broken.txt
[root@master1 data]# cat /var/data/broken.txt
dev/liveness-http
[root@master1 data]# 

[root@master1 data]# kubectl -n dev get events -o wide > /var/data/error.txt


[root@master1 data]# cat /var/data/error.txt
LAST SEEN   TYPE      REASON      OBJECT              SUBOBJECT                   SOURCE                             MESSAGE                                                         FIRST SEEN   COUNT   NAME
48m         Warning   Unhealthy   pod/liveness-http   spec.containers{liveness}   kubelet, workernode1.example.com   Liveness probe failed: HTTP probe failed with statuscode: 404   88m          61      liveness-http.17c3b533cec001d8
3m32s       Warning   BackOff     pod/liveness-http   spec.containers{liveness}   kubelet, workernode1.example.com   Back-off restarting failed container                            87m          378     liveness-http.17c3b53d95b09710




[root@master1 data]# kubectl get pods -n dev -w
NAME            READY   STATUS    RESTARTS   AGE
liveness-http   1/1     Running   0          13s
liveness-http   1/1     Running   1 (2s ago)   23s
liveness-http   1/1     Running   2 (2s ago)   41s
```
