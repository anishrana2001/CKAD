### You have rollout new pods "ckad0021-newpod" on your Kubernetes cluster under namespace ckad0021. Now, you need to streanten the network security. A network policy is already there. Your tasks are followed.

### - Update the new created pod "ckad0021-newpod" to allowed it to send and receive traffic from web and storage pods only.

### For your references: 
```

[root@master1 data]# kubectl -n ckad0021 get pods -o wide --show-labels 
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
ckad0021-newpod   1/1     Running   0          11s   172.16.14.114    workernode2.example.com   <none>           <none>            allow-access=true
storage           1/1     Running   0          11s   172.16.133.146   workernode1.example.com   <none>           <none>            app=secure-app
www               1/1     Running   0          12s   172.16.133.144   workernode1.example.com   <none>           <none>            app=secure-app
```
##
## 
```
[root@master1 data]# kubectl -n ckad0021 get netpol
NAME           POD-SELECTOR   AGE
default-deny   <none>         26s
```
##
##

```
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
```

## 
```
[root@master1 data]# kubectl -n ckad0021 exec -it ckad0021-newpod -- curl -o /dev/null -s -v -m 5 http://172.16.133.146 
*   Trying 172.16.133.146:80...
* Connection timed out after 5008 milliseconds
* Closing connection 0
command terminated with exit code 28
```
##
```
[root@master1 data]#  kubectl -n ckad0021 exec -it www -- curl -s -o /dev/null -m 5 -v http://172.16.14.114
*   Trying 172.16.14.114:80...
* Connection timed out after 5004 milliseconds
* Closing connection 0
command terminated with exit code 28
```
```
[root@master1 data]# kubectl -n ckad0021 get netpol/default-deny -o yaml > networkpolicy.yaml
```
```
[root@master1 data]# kubectl -n ckad0021 delete netpol/default-deny
networkpolicy.networking.k8s.io "default-deny" deleted
[root@master1 data]# 
```

```
[root@master1 data]# vi networkpolicy.yaml 
```
```
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
```
```
[root@master1 data]# kubectl apply -f networkpolicy.yaml
networkpolicy.networking.k8s.io/default-deny created

```
```
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
```
```
[root@master1 data]# kubectl -n ckad0021 get pods -o wide --show-labels
NAME              READY   STATUS    RESTARTS   AGE     IP               NODE                      NOMINATED NODE   READINESS GATES   LABELS
ckad0021-newpod   1/1     Running   0          7m11s   172.16.14.114    workernode2.example.com   <none>           <none>            allow-access=true
storage           1/1     Running   0          7m11s   172.16.133.146   workernode1.example.com   <none>           <none>            app=secure-app
www               1/1     Running   0          7m12s   172.16.133.144   workernode1.example.com   <none>           <none>            app=secure-app

```
```
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


## For question 2nd.

## Create lab.
```
[root@master1 data]# kubectl create ns kdpd002024
kubectl create deployment kdpd002024-deployment --image=nginx -n kdpd002024
namespace/kdpd002024 created
deployment.apps/kdpd002024-deployment created
```

### Solution start from here.
```

[root@master1 data]# kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".

[root@master1 data]# kubectl get deployments.apps -n kdpd002024
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
kdpd002024-deployment   1/1     1            1           2m24s

[root@master1 data]# kubectl -n kdpd002024 get pods --show-labels 
NAME                                     READY   STATUS    RESTARTS   AGE     LABELS
kdpd002024-deployment-6f96b4578d-2w42z   1/1     Running   0          2m28s   app=kdpd002024-deployment,pod-template-hash=6f96b4578d

[root@master1 data]# kubectl -n kdpd002024 edit deployments.apps kdpd002024-deployment 
deployment.apps/kdpd002024-deployment edited

[root@master1 data]# kubectl -n kdpd002024 get pods --show-labels 
NAME                                     READY   STATUS    RESTARTS   AGE   LABELS
kdpd002024-deployment-58c4ffb898-8ddmd   1/1     Running   0          10s   app=kdpd002024-deployment,pod-template-hash=58c4ffb898,role=webfrontend
kdpd002024-deployment-58c4ffb898-96wj6   1/1     Running   0          17s   app=kdpd002024-deployment,pod-template-hash=58c4ffb898,role=webfrontend
kdpd002024-deployment-58c4ffb898-cnklx   1/1     Running   0          10s   app=kdpd002024-deployment,pod-template-hash=58c4ffb898,role=webfrontend
kdpd002024-deployment-58c4ffb898-db9qp   1/1     Running   0          17s   app=kdpd002024-deployment,pod-template-hash=58c4ffb898,role=webfrontend
kdpd002024-deployment-58c4ffb898-gmp2p   1/1     Running   0          16s   app=kdpd002024-deployment,pod-template-hash=58c4ffb898,role=webfrontend


[root@master1 data]# kubectl -n kdpd002024 expose deployment kdpd002024-deployment --name=srv-kdpd002024 --port=8000 --target-port=80 --type=NodePort  --labels=role=webfrontend
service/srv-kdpd002024 exposed
[root@master1 data]# kubectl -n kdpd002024 get service/srv-kdpd002024 
NAME             TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
srv-kdpd002024   NodePort   10.100.188.221   <none>        8000:30400/TCP   16s


[root@master1 data]# kubectl -n kdpd002024 describe service/srv-kdpd002024 
Name:                     srv-kdpd002024
Namespace:                kdpd002024
Labels:                   role=webfrontend
Annotations:              <none>
Selector:                 app=kdpd002024-deployment
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.100.188.221
IPs:                      10.100.188.221
Port:                     <unset>  8000/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30400/TCP
Endpoints:                172.16.133.138:80,172.16.133.140:80,172.16.133.141:80 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

[root@master1 data]# kubectl -n kdpd002024 get endpoints
NAME             ENDPOINTS                                                           AGE
srv-kdpd002024   172.16.133.138:80,172.16.133.140:80,172.16.133.141:80 + 2 more...   27s
[root@master1 data]# 


[root@master1 data]# curl http://192.168.1.31:30400
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@master1 data]#
```


