
# How to create Liveness and Readiness probes manifest file for exam preparation.


## Create the NGINX configuration file. It should have two additional web pages, such as healthz and started.
```
cat <<EOF>> default.conf 
server {
    listen       80;
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

## Create Page for healthz endpoint

```
cat <<EOF>> healthz 
Server is up and running fine.
EOF
```
## Create Page for started endpoint 
```
cat <<EOF>> started 
Server is up and running fine.
EOF
```

## Now, create a configmaps so that we can import these files into pod manifest file.

```
kubectl create configmap healthz --from-file=healthz
kubectl create configmap started --from-file=started
kubectl create configmap nginx-conf --from-file=default.conf
```


## Create a POD. I did one mistake, so that we can reactify it. 
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
        - name: healthz
          mountPath: /usr/share/nginx/html/
        - name: started
          mountPath: /usr/share/nginx/html/
    livenessProbe:
      httpGet:
        path: /health
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 10
      failureThreshold: 1
    readinessProbe:
      httpGet:
        path: /started
        port: 80
      initialDelaySeconds: 2
      periodSeconds: 10
      failureThreshold: 1
  volumes:
  - name: config
    configMap:
      name: nginx-conf
  - name: healthz
    configMap:
      name: healthz
  - name: started
    configMap:
      name: started
EOF
```
