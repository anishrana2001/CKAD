

### Lab Preparation.
```
mkdir /var/data/
cd /var/data/
touch /var/data/pod.yaml /var/data/out1.json
kubectl create namespace kdp0024 
kubectl create namespace kdpd0023
kubectl -n kdpd0023 create deployment web --image=nginx:1.24.0
kubectl create ns qa 
kubectl create ns lab
kubectl create ns prod
kubectl create ns dev
kubectl create deployment failed-deploy --image=ngin
cat <<EOF>> liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
  namespace: dev
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
EOF
kubectl apply -f liveness-exec.yaml
```


# Question: 
#### Use context : `kubectl config use-context kubernetes-admin@kubernetes`
### Anytime a team needs to run a container on Kubernetes they will need to define a pod within which to run the container. Please complete the following:
###  - Create a POD manifest file "/var/data/pod.yaml" in YAML format with name of pod `app1` that runs a container named `green` using image `nginx:1.24.0` with these command line argument `-R 7 --testgreen`
###  - Create a pod from above yaml file created in previous step.
### -  when the pod is running, redirect summary data of pod into one file "/var/data/out1.json" in JSON format. Using kubectl command.
### - All the files are already created well in advance for you.



### Solution:
### Use the correct context:
```
kubectl config use-context kubernetes-admin@kubernetes
```
### We can create the POD from run command and redirect the output in the one given file.

```
kubectl run app1 --image=nginx:1.24.0 -o yaml --dry-run=client > /var/data/pod.yaml
```
#### Open the Kubernetes.io web page and search for "Command and Arguments"
### Now, modify the file so that we can insert other values.
```
vi /var/data/pod.yaml
```

#### For your references:
```
[root@master1 ~]# cat /var/data/pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app1
  name: app1
spec:
  containers:
  - image: nginx:1.24.0
    name: green                         # Modified
    resources: {}
    command: [ "/bin/echo" ]            # Add this line to print below values.
    args: ["-R", "7", "--testgreen"]    # Add this line.
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
[root@master1 ~]# 
```
###  Create the pod using this yaml file, this is our 2nd task.
```
kubectl create -f /var/data/pod.yaml
```
### Check the pod, it should be in running status.
```
kubectl get pods/app1 
```
### 3rd task of this question is to redirect the summary in  JSON  format on 1 file "/var/data/out1.json"

```
kubectl get pods/app1 -o json > /var/data/out1.json
```
#### For your references...
```
[root@master1 data]# kubectl get pods/app1 -w
NAME   READY   STATUS              RESTARTS   AGE
app1   0/1     ContainerCreating   0          11s
app1   1/1     Running             0          25s
app1   0/1     Completed           0          37s
app1   0/1     Completed           1 (16s ago)   40s
app1   0/1     CrashLoopBackOff    1 (12s ago)   52s
app1   0/1     Completed           2 (14s ago)   54s
^C
[root@master1 data]# kubectl logs pods/app1 
-R 7 --testgreen
[root@master1 data]# 
```

#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes
### Create a new deployment for running NGINX with the following parameters. 
#### - Run the deployment in the kdp0024 namespace. The namespace has already been created. 
#### - Name the deployment mydeploy and configure with 3 replicas
#### - Configure the pod with a container image of nginx:1.24.0
#### - Set an environment variable of NGINX_Port=8080 and also expose that port for the container above



### Solution : 

### use the correct context:
```
kubectl config use-context kubernetes-admin@kubernetes
```

### In exam, it would be good to use kubectl command to generate the object yaml file and the modify the parameters as per exam's question.
```
kubectl -n kdp0024  create deployment mydeploy  --replicas=3 --image=nginx:1.24.0 --port=8080  -o yaml --dry-run=client > mydeploy.yaml
```

#### Open the Kubernetes.io web page and search for "deployment env variable"

#### Now, open the deployment yaml file that we created in the previous step.
```
vi mydeploy.yaml
```
#### For your references:

```
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
        env:                                # Line Added
          - name: NGINX_Port                # Line Added
            value: "8080"                   # Line Added
status: {}
[root@master1 ~]# 
```
#### It's time to create the deployment
```
kubectl apply -f mydeploy.yaml
```
#### Our newly created deployment should be running state. 
```
kubectl -n kdp0024 get deployments.apps 
```

#### Pod should also be on running state.
```
kubectl -n kdp0024 get pods
```
#### For the post check, we can check the Enironment variable 
```
kubectl -n kdp0024 exec -it mydeploy-d44f485d8-948v2 -- printenv | grep NGINX_Port
```




# Question: 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes
### As a Kubernetes application developer you will often observe that sometimes you need to update the running application. 
### -  Update the web deployment in the kdpd0023 namespace with a maxSurge of 10% and a maxUnavailable of 5%
### -  Perform a rolling update of the web deployment changing the nginx:1.24.0 image version to 1.24.1
### -  Perform the rollback the web deployment to the previous version 


### Solution: 

### Use the correct context:
```
kubectl config use-context kubernetes-admin@kubernetes
```

### First, check the web deployment under the given namespace. 
```
kubectl -n kdpd0023 get deployments/web
```

#### Verify the maxSurge and maxUnavailable values.
```
kubectl -n kdpd0023 get deployments/web -o yaml | grep max
```

#### Now, we can edit the deployment with the help of "edit" command.
```
kubectl -n kdpd0023 edit deployment web 
```

#### Post check for task 1.
```
kubectl -n kdpd0023 get deployments/web -o yaml | grep max
```
#### Let's proceed for the tast 2.  Before update the image,  check the image name and its current version. 
```
kubectl -n kdpd0023 get deployments/web -o yaml | grep image
```


#### Now, Update the image of this deployment with the help of rolling update command.
Syntax
  kubectl set image (-f FILENAME | TYPE NAME) CONTAINER_NAME_1=CONTAINER_IMAGE_1 ... CONTAINER_NAME_N=CONTAINER_IMAGE_N		
```
kubectl -n kdpd0023 set image deployment web nginx=nginx:1.24.1
```

#### We can check the rollout history of this web deployment.
```
kubectl -n kdpd0023 rollout history deployment web
```
#### We can also check the image version.
```
kubectl -n kdpd0023 get deployments/web -o yaml | grep image
```

### Post checks are good, now we proceed for our last task i.e. rollback this upgrade.
```
kubectl -n kdpd0023 rollout undo deployment web
```
#### Post check for this rollback.
```
kubectl -n kdpd0023 rollout history deployment web
```

#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes

#### Developer occasionally need to submit pods that run periodically.
#### - Create a manifest file `/tmp/cronjob.yaml` in a YAML format.
#### - Yaml that runs the shell command "uname" in a single `busybox` container. 
#### - The command should run `every minute` and must complete within `28 seconds` or be terminated by Kubernetes. 
#### - The CronJob name and container name should both be `hellocron`
#### - Create the cronjob from the above manifest file.

### Solution:

#### If you are not aware about the cronjob syntax, then you can use the help command. Copy the first command, :-) 
```
kubectl create cronjob  -h
```

#### Now, we can copy the first example and udpate the command like belwo.
```
kubectl create cronjob hellocron --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- uname > /tmp/cronjob.yaml
```

### https://kubernetes.io/docs/concepts/workloads/controllers/job/#job-termination-and-cleanup
#### Open the file /tmp/cronjob.yaml
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: hellocron
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: hellocron
    spec:
      activeDeadlineSeconds: 28    ### 👈👈👈 Add this line "cronjob.spec.jobTemplate.spec.activeDeadlineSeconds"
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - uname
            image: busybox
            name: hellocron
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
```
### Create the cronjob from the yaml file.
```
kubectl apply -f /tmp/cronjob.yaml
```


#### Post check :
```
kubectl get cronjobs.batch 
```
#### Check the pods are created 
```
kubectl get pods
```

### Check the logs of this pod.
```
kubectl logs hellocron-29045640-hqr55
```

#### Full answer is below.
```
[root@master1 ~]# kubectl create cronjob -h | head
Create a cron job with the specified name.

Aliases:
cronjob, cj

Examples:
  # Create a cron job
  kubectl create cronjob my-job --image=busybox --schedule="*/1 * * * *"
  
  # Create a cron job with a command
[root@master1 ~]# 

[root@master1 ~]#  kubectl create cronjob hellocron --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- uname 
apiVersion: batch/v1
kind: CronJob
metadata:
  creationTimestamp: null
  name: hellocron
spec:
  jobTemplate:
    metadata:
      creationTimestamp: null
      name: hellocron
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - uname
            image: busybox
            name: hellocron
            resources: {}
          restartPolicy: OnFailure
  schedule: '*/1 * * * *'
status: {}
[root@master1 ~]#  kubectl create cronjob hellocron --image=busybox --schedule="*/1 * * * *" --dry-run=client -o yaml -- uname > /tmp/cronjob.yaml
[root@master1 ~]# vi /tmp/cronjob.yaml 
[root@master1 ~]# kubectl apply -f /tmp/cronjob.yaml 
[root@master1 ~]# kubectl apply -f /tmp/cronjob.yaml 
cronjob.batch/hellocron created
[root@master1 ~]# kubectl get cronjobs.batch 
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hellocron   */1 * * * *   False     1        3s              9s
[root@master1 ~]# kubectl get pods
NAME                       READY   STATUS      RESTARTS   AGE
hellocron-29045640-hqr55   0/1     Completed   0          12s
[root@master1 ~]# kubectl logs hellocron-29045640-hqr55 
Linux
[root@master1 ~]#
```
#  Question : 
#### Use context : kubectl config use-context kubernetes-admin@kubernetes

#### A deployment is failing on the cluster due an incorrect image being specified. You need to locate the deployment and fix the issue.


### Solution: 

### Use the correct context:
```
kubectl config use-context kubernetes-admin@kubernetes
```

#### Check the deployment on all namespaces. We can use "-A" option for all namespaces.
```
kubectl get deployment -A
```

#### Check the image of this failed deployment.
```
kubectl get deployments.apps failed-deploy -o yaml | grep -i image
```

#### From the above output, it is clear that failed-deploy deployment image is not set correctly. Let's correct it.
```
kubectl edit deployments.apps failed-deploy 
```

#### Perform the post check.
```
kubectl get  deployment -A
```

#  Question : 
#### Use context : `kubectl config use-context kubernetes-admin@kubernetes`

### A application is failing due to livenessProbe. If can be run on any of the below namespaces.
#### - qa
#### - lab
#### - prod
#### - dev

### Tasks need to be performed:
### Identify the broken pod and write its name and namespace to `/var/data/broken.txt` in the format `"namespace"/"pod"`
### Copy the events into the file /var/data/error.txt. Use `"-o wide"` output specifier with your command.
### Fix the issue.

### Solution: 

### Use the correct context:
```
kubectl config use-context kubernetes-admin@kubernetes
```
#### Check all the pods inside these namespaces.
```
kubectl get po -A | egrep "qa|prod|dev|lab"
```
#### First task is write faulty pod name and its namespace in one file "/var/data/broken.txt"
```
echo "dev/liveness-http" > /var/data/broken.txt
```

#### Post check!
```
cat /var/data/broken.txt
```

#### 2nd task is copy the event with "-o wide" option into the file "/var/data/error.txt"

```
kubectl -n dev get events -o wide > /var/data/error.txt
```

#### Post checks 
```
cat /var/data/error.txt
```
#### For 3rd task, we need to resolve this issue.  Redirect the manifest file into 1 file.

```
kubectl -n dev get pod/liveness-http -o  yaml > /var/data/liveness.yaml
```

#### Delete the faulty pod
```
kubectl -n dev delete pods/liveness-http
```

#### Edit the fautly pod yaml file.
```
vi /var/data/liveness.yaml
```
#### Issue is only with livenessprobs. Thus change the path of this pod.
```
spec:
  containers:
  - args:
    - /server
    image: registry.k8s.io/liveness
    imagePullPolicy: Always
    livenessProbe:
      failureThreshold: 3
      httpGet:
        httpHeaders:
        - name: Custom-Header
          value: Awesome
        path: /healthz              ## updated this line only.
        port: 8080
        scheme: HTTP
      initialDelaySeconds: 3
      periodSeconds: 3
      successThreshold: 1
      timeoutSeconds: 1
```

### Create the pod again.
```
kubectl apply -f /var/data/liveness.yaml
```
#### Post check:
```
kubectl get pods -n dev -w
```


## Clear the Lab.
```
kubectl delete deployments.apps/failed-deploy
kubectl delete -n kdp0024 deployments/mydeploy
kubectl delete  -n kdpd0023 deployments.apps/web
kubectl delete pods/app1
kubectl delete cronjobs.batch/hellocron
kubectl -n dev delete pods/liveness-http 
rm -rf /var/data/liveness-exec.yaml /var/data/pod.yaml /var/data/out1.json /var/data/liveness.yaml /var/data/error.txt /var/data/broken.txt
kubectl delete namespaces kdp0024 kdpd0023 qa prod dev lab 
```








