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
