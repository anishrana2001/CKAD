

```
kubectl create namespace ckad0021 
kubectl -n ckad0021 run www --image=nginx --labels=app=secure-app
kubectl -n ckad0021 run storage --image=nginx  --labels=app=secure-app
kubectl -n ckad0021 run ckad0021-newpod --image=nginx  --labels=allow-access=true
cat <<EOF>> default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: ckad0021
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
EOF

kubectl create -f default-deny.yaml
```


### You have rollout new pods "ckad0021-newpod" on your Kubernetes cluster under namespace ckad0021. Now, you need to streanten the network security. A network policy is already there. Your tasks are followed.

### - Update the new created pod "ckad0021-newpod" to allowed it to send and receive traffic from web and storage pods only.


### Check the running pods in the namespace "ckad0021 "
```
kubectl -n ckad0021 get pods -o wide --show-labels 
```

### Check the NetworkPolicy in this namespace
```
kubectl -n ckad0021 get netpol
```

### Describe networkPolicy
```
kubectl -n ckad0021 describe netpol/default-deny
```
### Check the Pods, if we can access another POD?
```
kubectl -n ckad0021 exec -it ckad0021-newpod -- curl -o /dev/null -s -v -m 5 http://172.16.133.143 
```
### Check the traffic on pod "ckad0021-newpod" 

```
 kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -m 5 -v http://ckad0021-newpod_IP 
```

### We can directly edit the NetworkPolicy but for exam point of view, don't take risk. Thus, perform these steps.
### Now, put the networkpolicy yaml file in one file.
```
kubectl -n ckad0021 get netpol/default-deny -o yaml > networkpolicy.yaml
```

### Delete the NetworkPolicy 
```
kubectl -n ckad0021 delete netpol/default-deny
```

### Edit the yaml file.
```
vi networkpolicy.yaml 
```

```
    matchLabels:
      allow-access: "true"


  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: secure-app
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: secure-app
```

### Now, create the network policy with new settings.
```
kubectl apply -f networkpolicy.yaml
```

### Verify the NetworkPolicy, now.
```
kubectl -n ckad0021 describe netpol default-deny 
```
### Check pods IPs
```
kubectl -n ckad0021 get pods -o wide --show-labels
```

### Send the incoming traffic to the pod "ckad0021-newpod" 
```
kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -v  http://ckad0021-newpod_IP
```




### Clear the lab for question 1.
```
kubectl -n ckad0021 delete pod/www  
kubectl -n ckad0021 delete pod/storage
kubectl -n ckad0021 delete pod/ckad0021-newpod
kubectl -n ckad0021 delete netpol/default-deny
kubectl delete namespace ckad0021
rm -rf default-deny.yaml
```




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
```
kubectl config use-context kubernetes-admin@kubernetes
```

### Check the deployment in the given namespace.
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
kubectl -n kdpd002024 delete service/srv-kdpd002024 
kubectl -n kdpd002024 delete deploy/kdpd002024-deployment
kubectl delete namespace kdpd002024
```


