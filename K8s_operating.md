# Operating application in k8s

## Deploying workloads
### Create deployment-nginx.yaml
```yaml
apiVersion: apps/v1
kind: deployment
  metadata:
  name: nginx-deployment
labels:
  app: nginx
spec:
  replicas: 2
  selector:
  matchLabels:
    app: nginx
```

```bash
kubectl apply -f deployment-nginx.yaml
```

### Verifying a deployment
- How to verify the status of deployment and troubleshoot it if needed
    - Confirm `successfully rolled out`
    ```bash
    kubectl rollout status deployment nginx-deployment
    ---
    deployment "nginx-deployment" successfully rolled out
    ```
    - Confirm `DESIRED` and `CURRENT` values is equal
    ```bash
    kubectl get deployments
    ---
    NAME            DESIRED CURRENT UP-TO-DATE AVAILABLE    AGE
    nginx-deployment    2    2      2           2           2m40s
    ```
    - Check RS and pods
    ```bash
    kubectl get rs, pods
    NAME                        DESIRED     CURRENT     READY   AGE
    nginx-deployment-5c689d88bb 2           2           2       28m
    NAME                                READY       STATUS      RESTARTS    AGE
    nginx-deployment-5c689d88bb-r2pp9   1/1         Running     0           28m
    nginx-deployment-5c689d88bb-xsc5f   1/1         Running     0           28m
    ```

### How to `edit`, `scale up`, `roll out` an new version using a ReplicaSet
#### Change from `image nginx 1.7.9` to `image nginx 1.16.0`
```bash
kubectl edit deployment nginx-deployment
```

#### Get rollout status 
```bash
kubectl rollout status deployment nginx-deployment
---
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
deployment "nginx-deployment"
```

#### Confirm spins up the new pods
```bash
kubectl get rs
---
NAME                        DESIRED     CURRENT  READY   AGE
nginx-deployment-5c689d88bb 0           0        0       36m
nginx-deployment-f98cbd66f  2           2        2       46s
```

#### How to create change cause annotation
```bash
kubectl annotate deployment nginx-deployment
---
kubernetes.io/change-cause="image updated to 1.16.0"
```

### How to `rollback` a deployment
- Comparing the annotations and rollback the deployment to an older revision when needed
    - Check the details and events for the deployment
    ```bash
    kubectl describe deployments
    ```
    - Display the rollout history for the deployment
    ```bash
    kubectl rollout history deployment nginx-deployment
    ---
    deployment.extensions/nginx-deployment
    REVISION    CHANGE-CAUSE
    1           <none>
    2           image updated to 1.16.0
    3           image updated to 1.17.0 and scaled up to 3 replicas
    ```
    - `Rollback` the last rollout
    ```bash
    kubectl rollout undo deployment nginx-deployment
    ```
    - `Rollback` to a specifc revision
    ```bash
    kubectl rollout undo deployment nginx-deployment --to-revision=1
    ```

## How to use kubernetes operators
- An operators helps to remove manual steps, application-specific preparation and post-deployment steps and even automates second-day operations such as scaling or upgrading them

### Installing `KUDO` (Kubernetes Universal Declarative Operator) and KUDO kubectl plugin
```bash
# install krew
https://krew.sigs.k8s.io/docs/user-guide/setup/install/

(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/krew.tar.gz" &&
  tar zxvf krew.tar.gz &&
  KREW=./krew-"${OS}_${ARCH}" &&
  "$KREW" install krew
)

export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

helm repo add jetstack https://charts.jetstack.io

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.2.0 \
  --create-namespace \
  --set installCRDs=true
  
kubectl krew install kudo
```