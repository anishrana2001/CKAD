# **CANARY Deployment**

#
-  _**Will create 2 Deployments**_ 
- _**attach the same label to both deployments**_
- _**create 1 service and point to both deployments with the help of "same" label.**_
---
---
### Task 1:
### - Create a namespace called `tiger` namespace.
### - Create a deployment called `blue` with `6 replicas`, using the nginx image `1.26.3` inside the `tiger` namespace.
### Label the pods `app=blue` and `tier=web`
### - Expose `port 80` for the nginx containers.
### Task 2:
### - Create a Service called `web-srv` to route traffic to `blue`.
### Task 3:
### - Create an identical Deployment named canary-green-deployment, in the same namespace.
- Modify the Deployment so that:
- A maximum number of 10 pods run in the tiger namespace.
- 40% of the `web-srv` service's traffic goes to the `canary-green-deployment`.

## Solution
### Task 1:
### - Create a namespace called `tiger` namespace.
```
kubectl create namespace tiger
```
### - Create a deployment called `blue` with `6 replicas`, using the nginx image `1.26.3` inside the `tiger` namespace.
### - Expose `port 80` for the nginx containers.
```
kubectl create deployment blue --image=nginx:1.26.3 -n tiger --port=80 --replicas=6 --dry-run=client -o yaml > /tmp/blue.yaml
```
### Label the pods `app=blue` and `tier=web`

vi /tmp/blue.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: blue
  name: blue
  namespace: tiger
spec:
  replicas: 6
  selector:
    matchLabels:
      app: blue
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: blue
        tier: web # 👈👈👈 Add label here under template section
    spec:
      containers:
      - image: nginx:1.26.3
        name: nginx
        ports:
        - containerPort: 80
        resources: {}
status: {}
```
### Post checks for Task 1
```
kubectl -n tiger get deployment
```
```
kubectl -n tiger  get pods --show-labels 
```
```
kubectl -n tiger  get pods -l tier=web
```
### Task 2:
### - Create a Service called `web-srv` to route traffic to `blue` and select the label tier=web

```
kubectl -n tiger expose deployment blue --port=80 --target-port=80 --name=web-srv --dry-run=client -o yaml > /tmp/web-srv.yaml
```
```
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: blue
  name: web-srv
  namespace: tiger
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    tier: web  # 👈👈👈 Add label here under 
status:
  loadBalancer: {}
```
#### Create the service
```
kubectl apply -f  /tmp/web-srv.yaml 
```

#### Post checks for Task 2:
```
kubectl -n tiger get service
```
```
kubectl -n tiger describe service/web-srv 
```
### Task 3:
### - Create an identical Deployment named canary-green-deployment, in the same namespace.
- Modify the Deployment so that:
- A maximum number of 10 pods run in the tiger namespace.
- 40% of the `web-srv` service's traffic goes to the `canary-green-deployment`.

```
kubectl -n tiger get deployments.apps blue -o  yaml > /tmp/blue-tiger.yaml
```
```
vi /tmp/blue-tiger.yaml
```

- **change the name of deployment to `canary-green-deployment`.**

```
kubectl apply -f /tmp/blue-tiger.yaml
```
```
kubectl -n tiger  get pods -l tier=web
```
#### One must observe 12 pods IPs when describe the service `web-srv `.
```
kubectl -n tiger describe service/web-srv 
```
```
kubectl get pods -n tiger 
kubectl get pods -n tiger | grep -v NAME | wc -l
```
### As per the task 
- A maximum number of 10 pods run in the tiger namespace.
- 40% of the `web-srv` service's traffic goes to the `canary-green-deployment`.

#### Blue deployment = 6 Pods
#### canary-green-deployment = 4 Pods
#### If we make the changes like this, then 10 Pods will be there and 40% traffic will be served by canary-green-deployment. 

### Modify the replicas from 6 to 4
```
kubectl -n tiger  edit deployments.apps canary-green-deployment
```
### Post checks for Task 3:
```
kubectl get pods -n tiger | grep -v NAME | wc -l
```
```
kubectl -n tiger  get pods -l tier=web
```

## How to clear the LAB?
```
kubectl -n tiger delete service/web-srv deployments.apps/blue deployments.apps/canary-green-deployment 
```
```
kubectl delete namespaces tiger 
```
