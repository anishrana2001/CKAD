

```
kubectl create namespace ckad0021
kubectl run www -image=nginx -n ckad0021
kubectl run storage --image=nginx -n ckad0021
kubectl run kdpd002021-newpod --image=nginx
```

```yaml
cat <<EOF>> example-network-policy1
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-network-policy1
  namespace: ckad0021
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  - Egress
  ingress:                          # Incomming traffic towards pod.
    - from:
        - podSelector:
            matchLabels:
              allow-access: "true"  # Accept only incoming traffic which has this label.
  egress:                           # Out going traffic from Pod
    - to:
        - podSelector:
            matchLabels:
              allow-access: "true"  # Out going traffic to only Pods which has this label.
EOF
```


### You have rollout new pods "ckad0021-newpod" on your Kubernetes cluster under namespace ckad0021. Now, you need to streanten the network security. A network policy is already there. You tasks are followed.

### - Update the new created pod "ckad0021-newpod" to allowed it to send and receive traffic from web and storage pods only.


