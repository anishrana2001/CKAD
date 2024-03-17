# Topic: Application Environment, Configuration and Security  25%
#
## Discover and use resources that extend Kubernetes (CRD, Operators)
## Understand authentication, authorization and admission control
## Understand requests, limits, quotas
## Understand ConfigMaps
## Define resource requirements
## Create & consume Secrets
## Understand ServiceAccounts
## Understand Application Security (SecurityContexts, Capabilities, etc.)
##
##
##############################################################################################
## You will probably get question from these topics.                                         #
### Understand requests, limits, quotas                                                      #
### Understand requests, limits, quotas                                                      #
### Understand ConfigMaps                                                                    #
### Create & consume Secrets                                                                 #
### Understand ServiceAccounts                                                               #
##############################################################################################
### 
### 
### If you wish, you can create this lab on your own VMs or Laptops.
### My Cluster config, for your information only.
```yaml
[root@master1 data]# kubectl config view 
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://192.168.1.31:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes                       #### This is my Cluster name.
    user: kubernetes-admin                    #### This is my Kubernetes user name.
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

### How to create context for user, for exam purpose ?
```
kubectl config set-context ek8s \
--namespace=project-one \
--namespace=project-production \
--cluster=kubernetes \
--user=kubernetes-admin
```
###############################################################

### Question:     kubectl config use-context ek8s

### Use: kubectl config use-context ek8s

### Create a pod that requests a certain amount of CPU and memory.
### Complete the task:
### - Create a namespace  project-one
### - Create a POD named nginx-resources in the project-one namespace that required a minimum of 200m CPU and 1Gi memory for its container
### - The pod should use the  nginx image.  

### Solution :

### 
```
kubectl config use-context ek8s
```
```
kubectl create ns project-one
```
### POD yaml file.

```yaml
cat <<EOF>> q3-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-resources
  namespace: project-one
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "1Gi"
        cpu: "200m"
EOF
```

```
kubectl create -f q3-pod.yaml 
```

### Check if our newly created pod is running?
```
kubectl get pod -n project-one
```
### Post check, we can execute the command "kubectl describe" to see the CPU and memory information.
```
kubectl -n project-one describe pod/nginx-resources | grep -A 2 -i Requests
```
#### First question end here.
###                                                                                            .
###                                                                                            .
### 2nd Qustion: Use: kubectl config use-context ek8s
### You are tasked to create a ConfigMap and consume the ConfigMap in a pod using a volume mount. 
### Task need to be completed: 
### - Create a configMap named another-config containing the key/value pair. key7/value5.
### - Create a pod named nginx-configmap  and should have nginx image and mount the key you just created into the pod under directory /data/config 
### 
### Solution:

```
kubectl config use-context ek8s
```
###  Open the https://kubernetes.io/ , click on documentation , search configMap

```
kubectl create configmap another-config --from-literal=key7=value5
```

```yaml
cat <<EOF>> q4.pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-configmap
spec:
  containers:
  - name: mypod
    image: nginx
    volumeMounts:
    - name: foo
      mountPath: "/data/config"
  volumes:
  - name: foo
    configMap:
      name: another-config
EOF
```

### Create the pod.
```
kubectl create -f q4.pod.yaml 
```

```
kubectl get po 
```
### Post checks: 
```
kubectl exec -it pods/nginx-configmap -- cat /data/config/key7 ; echo
```
###                 .
###                 .
### 3: Question: 
### Use: kubectl config use-context ek8s
### You are tasked to create a secret and consume the secret in a pod using environment variables as follows
### - Create a secret name app-secret1 with a key/value pair.  key30/value4
### - Start a nginx POD named nginx-secret1 using container image nginx and add an environment a variable exposing the value of  the secret key key30 using BEST_VARIABLE1 as the name of the environment variable inside the pod. 

```
kubectl config use-context ek8s
```
```
kubectl create secret generic app-secret1 --from-literal=key30=value4
```

###  https://kubernetes.io/ , click on documentation , search secret env

```yaml
cat <<EOF>> q2-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secret1
spec:
  containers:
  - name: mycontainer
    image: nginx
    env:
      - name: BEST_VARIABLE1
        valueFrom:
          secretKeyRef:
            name: app-secret1
            key: key30
EOF
```
```
kubectl create -f q2-pod.yaml
```
```
kubectl get pods/nginx-secret1
```
```
kubectl exec -it pods/nginx-secret1 -- printenv | grep BEST
```

###                            .
###                            .
### 4: Question:
### Use: kubectl config use-context ek8s
### There is one deployment web-app is running under namespace project-production
### Task need to be performed:
### - Update the web-app deployment to run as an app serviceaccount. 
### - This serviceaccount is already created.
###

### Create the lab for this question:
```
kubectl create ns project-production
kubectl create deployment web-app --image=nginx  -n project-production
kubectl create sa app -n project-production
```
### Solution: 
``` 
kubectl config use-context ek8s
``` 

### https://kubernetes.io/ , click on documentation , search serviceAccount

``` 
kubectl -n project-production describe deployments/web-app | grep -i Service
``` 

``` 
kubectl edit deployment web-app -n project-production
```
### Add this values: "Service Account:  app" on step 5 , see the below print screen.

![image](https://github.com/anishrana2001/CKAD/assets/93471182/cd858df6-9ebb-4bc9-bf48-81e9e690a225)


### Post Checks:
```
kubectl -n project-production describe deployments/web-app | grep Service
```
## Clear the lab.
```
kubectl config use-context kubernetes-admin@kubernetes
kubectl -n project-one delete pod/nginx-resources
kubectl delete namespaces project-one
kubectl delete -f q4.pod.yaml
kubectl delete configmaps another-config
kubectl delete pod/nginx-secret1
kubectl delete secrets app-secret1
kubectl -n project-production delete deployment/web-app
kubectl -n project-production delete sa/app
kubectl delete namespaces project-production
rm -rf q3-pod.yaml q4.pod.yaml q2-pod.yaml
```
