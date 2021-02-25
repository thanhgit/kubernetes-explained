# Kubernetes volume
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

## Assign openebs namespace to the current context
```bash
kubectl config set-context $(kubectl config current-context) -n openebs
```