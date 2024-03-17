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
        memory: “1Gi"
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


