# Question from Implement probes and health checks
#
## A pod is running on the cluster, but it is not responding.
## It is expected to Kubernetes to restart the pod when an endpoint returns an http 500 on the /healthz endpoint. The service, liveness-http, should never send traffic to the pod when it is falling. 
## Please complete the following:
## - The application has an endpoint. /started. That will indicate if it can accept traffic by returning HTTP 200. If the endpoint returns an HTTP 500, the application has not yet finished initialization.
## 
## - The application has another endpoint. /healthz that will indicate if the application is still working as expected by returning an HTTP 200 if the endpoint returns an HTTP 500 the application is no longer responsive.
## 
## - Configure the liveness-http pod provided to use these endpoints.
## - The probes should use port 8080
## SOlution:
###
### In order to reslove the issue, first, we have to create a lab for this question. In this question, we have two endpoints. Thus, we have to modify the NGINX configuration file with these two endpoints.
### For these 2 points, we must create 2 files. So, that we can send the request to these pages?




### Create the NGINX configuration file. It should have two additional web pages, such as healthz and started.
```
cat <<EOF>> default.conf 
server {
    listen       8080;      #  The probes should use port 8080
    listen  [::]:80;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

server { 
       location /healthz {
        access_log off;
        return 200;
		}
	}
server { 
       location /started {
        access_log off;
        return 200;
		}
	}
EOF
```

### Create Page for healthz endpoint

```
cat <<EOF>> healthz 
healthz endpoint is up and running fine.
EOF
```
### Create Page for started endpoint 
```
cat <<EOF>> started 
started endpoint is up and running fine.
EOF
```

### Now, create a configmaps so that we can import these files into pod manifest file.

```
kubectl create configmap healthz --from-file=healthz
kubectl create configmap started --from-file=started
kubectl create configmap nginx-conf --from-file=default.conf
```

### How to create Liveness and Readiness probes manifest file for exam preparation.
### Create a POD. I did one mistake, so that we can reactify it. 
```yaml
cat <<EOF>> liveness-readiness-http.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: readiness
  name: live-readi-http
spec:
  containers:
  - name: live-readi-container
    image: nginx
    volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
        - name: healthz-started
          mountPath: /usr/share/nginx/html/
    livenessProbe:              # If application works, then it should return HTTP 200. If the endpoint returns an HTTP 500 the application is no longer responsive.
      httpGet:                  # If it is asked us to check http return code then we should use httpGet method.
        path: /healthz          # The application has an endpoint /healthz. 
        port: 8080              # The probes should use port 8080
      initialDelaySeconds: 2
      periodSeconds: 10
      failureThreshold: 3
    readinessProbe:             # If it can accept traffic by returning HTTP 200 
      httpGet:                  # HTTP 200 only comes, when we use httpGet method.
        path: /started          # /started 
        port: 8080              # The probes should use port 8080
      initialDelaySeconds: 5
      periodSeconds: 10
      failureThreshold: 3
  volumes:
  - name: config
    configMap:
      name: nginx-conf
  - name: healthz-started
    projected:
      sources:
      - configMap:
          name: healthz
          items:
            - key: healthz
              path: healthz
      - configMap:
          name: started
          items:
            - key: started
              path: started
EOF
```

```
kubectl apply -f liveness-readiness-http.yaml
```

```
kubectl get pods/live-readi-http -w 
```

### Or  you can check the pods describe command.

```
kubectl describe pod/live-readi-http
```
```
kubectl get pod liveness-http -o yaml > q6-pod.yaml
```

