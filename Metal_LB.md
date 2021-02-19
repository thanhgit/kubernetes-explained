# Install Metal LB

## Change config map
```bash
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

## Apply the manifest
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```
- new-subnet.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.100-192.168.1.250 #Update this with your Nodes IP range 
```
- Apply
```bash
kubectl apply -f new-subnet.yaml
---
NAMESPACE        NAME                             READY   STATUS    RESTARTS   AGE
metallb-system   controller-65db86ddc6-g7tkt      1/1     Running   0          2m8s
metallb-system   speaker-g5hbw                    1/1     Running   0          2m8s
metallb-system   speaker-m85wv                    1/1     Running   0          2m8s
metallb-system   speaker-mwlhr                    1/1     Running   0          2m8s
```

## Testing
- nginx-demo.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    name: http
```
- Apply
```bash
kubectl apply -f nginx-demo.yaml
```
- Test
```bash
kubectl get svc
---
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1       <none>          443/TCP        90m
nginx        LoadBalancer   10.105.129.46   192.168.1.100   80:32017/TCP   106s
```