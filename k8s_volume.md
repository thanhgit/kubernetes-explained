# Kubernetes volume - OpenEBS
- Need for distributed storage in any cloud environment is complex
- Dynamic provisioning is a feature for many cluster solutions (For example: PersistentVolume, which reserves persistent, reclaimable, local storage for processes)
- StorageClass are a prerequisite for dynamic provisioning. There are several knobs in the StorageClass API definition that allow you to communicate preferred storage to a kubernetes cluster 
  - StorageClass is an interface for defining storage requirements for a pod - not an implemention 
  - StorageClasses are declarative, PersistentVolumes are imperative
  - PersistentVolumeClaim can be fullfilled without a StorageClass 
  - StorageClasses and dynamic storage enable commoditization of storage for many apps, ...
## Enabling dynamic provisioning
- For HDD
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
```
- For SSD
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

## OpenEBS
- OpenEBS adopts Container Attached Storage (CAS) approach, where each workload is provided with a dedicated storage controller
- OpenEBS implements granular storage policies and isolation that enable users to optimize storage for each specific workload

## Install OpenEBS
- Prerequisite: adding iSCSI support
```bash
sudo apt-get update
sudo apt-get install open-iscsi
sudo service open-iscsi restart
```

## Get Kubernetes context
```bash
kubectl config current-context
```

## Create kubernetes admin
```bash
kubectl config set-context admin-ctx --cluster=kubernetes --user=kubernetes-admin
---
Context "admin-ctx" created.
```
## Using admin-ctx
```bash
kubectl config use-context admin-ctx
```
## Assign openebs namespace to the current context
```bash
kubectl config set-context $(kubectl config current-context) -n openebs
```

## Install with helm
```bash
helm install --namespace openebs --name openebs stable/openebs --version 1.4.0
```

## Install with kubecl
```bash
kubectl apply -f https://openebs.github.io/charts/openebs-operator-1.4.0.yaml
```

### Verifying installation
```bash
kubectl get pods -n openebs | wc -l
---
9
```
- Consists of openebs-ndm is a daemon set in all nodes, openebs-provisioner, maya-apiserver and openebs-snapshot-operator

### Verifying storage class
```bash
kubectl get sc | wc -l
---
6
```

### Verifying block device
```bash
kubectl get blockdevice -n openebs
```

### Verifying Jiva defaul pool - default
```bash
kubectl get sp
---
NAME      AGE
default   5d22h
```

### Assign label `node=openebs` for storage nodes
```bash
kubectl label nodes server1 node=openebs
kubectl label nodes server2 node=openebs
kubectl label nodes master node=openebs
```

## Common errors
```bash
Error from server (InternalError): error when creating "demo-percona-mysql-pvc.yaml": Internal error occurred: failed calling webhook "admission-webhook.openebs.io": Post "https://admission-server-svc.openebs.svc:443/validate?timeout=5s": context deadline exceeded
---
kubectl edit validatingwebhookconfigurations  openebs-validation-webhook-cfg

failurePolicy: Ignore 
```