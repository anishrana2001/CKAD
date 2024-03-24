## Question 2: 
### Lab creation ###
```
kubectl create ns kdpd002024
kubectl create deployment kdpd002024-deployment --image=nginx -n kdpd002024
```


### Use: kubectl config use-context kubernetes-admin@kubernetes
### Your task is scaling an existing deployment for availability and creating a service to expose the deployment within your infrastructure.  
### Start with the deployment named kdpd002024-deployment which has already been deployed to the namespace kdpd002024 edit it to :
### .
### - Add the role: webfrontend  key/value label to the pod template metadata to identify the pod for the service definition.
### - have 5 Replicas

### Next, create and deploy in namespace kdpd002024 a service that accomplished the following:
### - Exposes the services on TCP port 8000
### - is mapped to the pods defined by the specification of kdpd002024-deployment
### - should have NodePort type.
### - has a name of srv-kdpd002024.



### use the right context.

### Check the deploy in the given namespace.
```
kubectl get deployments.apps -n kdpd002024
```

### Check the Pods, with labels.
```
kubectl -n kdpd002024 get pods --show-labels 
```

### First thing first, take the backup of this deployment into one file.
```
kubectl -n kdpd002024 get  deployments.apps kdpd002024-deployment -o yaml > kdpd002024-deployment.yaml
```

### Now, edit this deployment and increase the replicas and add one more label at pod template.
```
kubectl -n kdpd002024 edit deployments.apps kdpd002024-deployment 
```


### For post check, we can check the PODs labels.
```
kubectl -n kdpd002024 get pods --show-labels 
```

### We can create a service as per question. 
```
kubectl -n kdpd002024 expose deployment kdpd002024-deployment --name=srv-kdpd002024 --port=8000 --target-port=80 --type=NodePort  --labels=role=webfrontend
```

### Check again the pods' labels. Its should get the new labels
```
kubectl -n kdpd002024 get pods --show-labels 
```
### Check the IP address of our newly created service.
```
kubectl -n kdpd002024 get service/srv-kdpd002024 
```


### For more details, use describe command.
```
kubectl -n kdpd002024 describe service/srv-kdpd002024 
```
###  If you wish, you can check the endpoints. Not required in the EXAM.
```
kubectl -n kdpd002024 get endpoints
```
### For post check, you can curl the VM IP address with port number. 
```
curl http://192.168.1.31:31105
```


### Clear the LAB for question 2.
```
kubectl -n kdpd002024 delete srv/srv-kdpd002024
kubectl -n kdpd002024 delete deploy/kdpd002024-deployment
kubectl delete namespace kdpd002024
```
