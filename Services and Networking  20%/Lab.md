

```
kubectl create namespace ckad0021 
kubectl -n ckad0021 run www --image=nginx --labels=app=secure-app
kubectl -n ckad0021 run storage --image=nginx  --labels=app=secure-app
kubectl -n ckad0021 run ckad0021-newpod --image=nginx  
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

### Check the traffic on pod "ckad0021-newpod" 

```
 kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -m 5 -v http://ckad0021-newpod_IP 
```

### Now, put the yaml file in one file.
```
kubectl -n ckad0021 get netpol/default-deny -o yaml > networkpolicy.yaml
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






### For your references: 
```

[root@master1 data]# kubectl -n ckad0021 get pods -o wide --show-labels 
NAME              READY   STATUS    RESTARTS   AGE     IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
ckad0021-newpod   1/1     Running   0          2m      172.16.133.140   workernode1.example.com   <none>           <none>            run=ckad0021-newpod
storage           1/1     Running   0          6h21m   172.16.14.111    workernode2.example.com   <none>           <none>            app=secure-app
www               1/1     Running   0          6h23m   172.16.133.136   workernode1.example.com   <none>           <none>            app=secure-app
[root@master1 data]# kubectl -n ckad0021 get netpol
NAME           POD-SELECTOR   AGE
default-deny   <none>         21s
[root@master1 data]# kubectl -n ckad0021 describe netpol/default-deny 
Name:         default-deny
Namespace:    ckad0021
Created on:   2024-03-23 19:22:26 +0530 IST
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     <none> (Allowing the specific traffic to all pods in this namespace)
  Allowing ingress traffic:
    <none> (Selected pods are isolated for ingress connectivity)
  Allowing egress traffic:
    <none> (Selected pods are isolated for egress connectivity)
  Policy Types: Ingress, Egress
[root@master1 data]# kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -v http://172.16.133.140
^Ccommand terminated with exit code 130
[root@master1 data]# kubectl -n ckad0021 get netpol/default-deny -o yaml > networkpolicy.yaml
[root@master1 data]# vi networkpolicy.yaml 

---------------
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"networking.k8s.io/v1","kind":"NetworkPolicy","metadata":{"annotations":{},"name":"default-deny","namespace":"ckad0021"},"spec":{"podSelector":{},"policyTypes":["Ingress","Egress"]}}
  creationTimestamp: "2024-03-23T13:52:26Z"
  generation: 1
  name: default-deny
  namespace: ckad0021
  resourceVersion: "268014"
  uid: e9501592-74e2-4576-bd73-b7b0ce6c7c7c
spec:
  podSelector:
    matchLabels:
      allow-access: "true"
  policyTypes:
  - Ingress
  - Egress
  ingress:                          # Incomming traffic towards pod.
    - from:
        - podSelector:
            matchLabels:
              app: secure-app        # Accept only incoming traffic which has this label.
  egress:                            # Out going traffic from Pod
    - to:
        - podSelector:
            matchLabels:
              app: secure-app        # Out going traffic to only Pods which has this label.
--------------------------------


[root@master1 data]# kubectl -n ckad0021 describe netpol default-deny 
Name:         default-deny
Namespace:    ckad0021
Created on:   2024-03-23 19:35:43 +0530 IST
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
[root@master1 data]# kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -v  http://172.16.133.140
*   Trying 172.16.14.111:80...
* Connected to 172.16.14.111 (172.16.14.111) port 80 (#0)
> GET / HTTP/1.1
> Host: 172.16.14.111
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.25.4
< Date: Sat, 23 Mar 2024 14:08:25 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Wed, 14 Feb 2024 16:03:00 GMT
< Connection: keep-alive
< ETag: "65cce434-267"
< Accept-Ranges: bytes
< 
{ [615 bytes data]
* Connection #0 to host 172.16.14.111 left intact
[root@master1 data]#
```
