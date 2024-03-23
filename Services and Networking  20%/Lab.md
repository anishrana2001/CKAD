

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


### You have rollout new pods "ckad0021-newpod" on your Kubernetes cluster under namespace ckad0021. Now, you need to streanten the network security. A network policy is already there. You tasks are followed.

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




### Clear the lab.
```
kubectl -n ckad0021 delete pod/www  
kubectl -n ckad0021 delete pod/storage
kubectl -n ckad0021 delete pod/ckad0021-newpod
kubectl -n ckad0021 delete netpol/default-deny
kubectl delete namespace ckad0021
rm -rf default-deny.yaml
```

### For your references: 
```

[root@master1 data]# kubectl -n ckad0021 get pods -o wide --show-labels 
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
ckad0021-newpod   1/1     Running   0          11s   172.16.14.114    workernode2.example.com   <none>           <none>            allow-access=true
storage           1/1     Running   0          11s   172.16.133.146   workernode1.example.com   <none>           <none>            app=secure-app
www               1/1     Running   0          12s   172.16.133.144   workernode1.example.com   <none>           <none>            app=secure-app
[root@master1 data]# kubectl -n ckad0021 get netpol
NAME           POD-SELECTOR   AGE
default-deny   <none>         26s
[root@master1 data]# kubectl -n ckad0021 describe netpol/default-deny
Name:         default-deny
Namespace:    ckad0021
Created on:   2024-03-23 21:08:47 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    <none> (Selected pods are isolated for ingress connectivity)
  Allowing egress traffic:
    <none> (Selected pods are isolated for egress connectivity)
  Policy Types: Ingress, Egress
[root@master1 data]# kubectl -n ckad0021 exec -it ckad0021-newpod -- curl -o /dev/null -s -v -m 5 http://172.16.133.146 
*   Trying 172.16.133.146:80...
* Connection timed out after 5008 milliseconds
* Closing connection 0
command terminated with exit code 28
[root@master1 data]#  kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -m 5 -v http://172.16.14.114
*   Trying 172.16.14.114:80...
* Connection timed out after 5004 milliseconds
* Closing connection 0
command terminated with exit code 28
[root@master1 data]# kubectl -n ckad0021 get netpol/default-deny -o yaml > networkpolicy.yaml

[root@master1 data]# kubectl -n ckad0021 delete netpol/default-deny
networkpolicy.networking.k8s.io "default-deny" deleted
[root@master1 data]# 

[root@master1 data]# vi networkpolicy.yaml 

[root@master1 data]# cat  networkpolicy.yaml 
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: "2024-03-23T15:38:47Z"
  generation: 1
  name: default-deny
  namespace: ckad0021
  resourceVersion: "277203"
  uid: 973e0fc3-39c4-4388-9a31-0584606b2abb
spec:
  podSelector:
    matchLabels:
      allow-access: "true"
  policyTypes:
  - Ingress
  - Egress

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
[root@master1 data]# 

[root@master1 data]# kubectl apply -f networkpolicy.yaml
networkpolicy.networking.k8s.io/default-deny created
[root@master1 data]# kubectl -n ckad0021 describe netpol default-deny 
Name:         default-deny
Namespace:    ckad0021
Created on:   2024-03-23 21:15:45 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     allow-access=true
  Allowing ingress traffic:
    To Port: <any> (traffic allowed to all ports)
    From:
      PodSelector: app=secure-app
  Allowing egress traffic:
    To Port: <any> (traffic allowed to all ports)
    To:
      PodSelector: app=secure-app
  Policy Types: Ingress, Egress
[root@master1 data]# kubectl -n ckad0021 get pods -o wide --show-labels
NAME              READY   STATUS    RESTARTS   AGE     IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
ckad0021-newpod   1/1     Running   0          7m11s   172.16.14.114    workernode2.example.com   <none>           <none>            allow-access=true
storage           1/1     Running   0          7m11s   172.16.133.146   workernode1.example.com   <none>           <none>            app=secure-app
www               1/1     Running   0          7m12s   172.16.133.144   workernode1.example.com   <none>           <none>            app=secure-app
[root@master1 data]# kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -v  http://172.16.14.114
*   Trying 172.16.14.114:80...
* Connected to 172.16.14.114 (172.16.14.114) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.16.14.114
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.25.4
< Date: Sat, 23 Mar 2024 15:46:13 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Wed, 14 Feb 2024 16:03:00 GMT
< Connection: keep-alive
< ETag: "65cce434-267"
< Accept-Ranges: bytes
< 
{ [615 bytes data]
* Connection #0 to host 172.16.14.114 left intact
[root@master1 data]# 
```
