# Administer a cluster

## Check version
```bash
kubeadm version
---
kubeadm version: &version.Info{Major:"1", Minor:"20", GitVersion:"v1.20.2", GitCommit:"faecb196815e248d3ecfb03c680a4507229c2a56", GitTreeState:"clean", BuildDate:"2021-01-13T13:25:59Z", GoVersion:"go1.15.5", Compiler:"gc", Platform:"linux/amd64"}
```

## Check upgrade plan
```bash
kubeadm upgrade plan
---
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.20.2
[upgrade/versions] kubeadm version: v1.20.2
[upgrade/versions] Latest stable version: v1.20.2
[upgrade/versions] Latest stable version: v1.20.2
[upgrade/versions] Latest version in the v1.20 series: v1.20.2
[upgrade/versions] Latest version in the v1.20 series: v1.20.2
```

## Upgrade new version
```bash
sudo kubeadm upgrade apply v1.20.x
```
- References: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#how-it-works

## Upgrade node
```bash
kubeadm upgrade node
---
[upgrade] Reading configuration from the cluster...
[upgrade] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade] Upgrading your Static Pod-hosted control plane instance to version "v1.20.2"...
Static pod: kube-apiserver-master hash: 9dd749e3711b8317807cda6ed275c71f
Static pod: kube-controller-manager-master hash: 9df73789607ffc47525cb22758f8c9df
Static pod: kube-scheduler-master hash: 69cd289b4ed80ced4f95a59ff60fa102
[upgrade/etcd] Upgrading to TLS for etcd
Static pod: etcd-master hash: 27072eb2438cc850e1506536264fef46
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Current and new manifests of etcd are equal, skipping upgrade
[upgrade/etcd] Waiting for etcd to become available
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests673315340"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Current and new manifests of kube-apiserver are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-controller-manager" upgrade
[upgrade/staticpods] Current and new manifests of kube-controller-manager are equal, skipping upgrade
[upgrade/staticpods] Preparing for "kube-scheduler" upgrade
[upgrade/staticpods] Current and new manifests of kube-scheduler are equal, skipping upgrade
[upgrade] The control plane instance for this node was successfully updated!
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[upgrade] The configuration for this node was successfully updated!
[upgrade] Now you should go ahead and upgrade the kubelet package using your package manager.

```
## Check certificate expiration
```bash
kubeadm certs check-expiration
---
ERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Feb 08, 2022 10:25 UTC   363d                                    no      
apiserver                  Feb 08, 2022 10:25 UTC   363d            ca                      no      
apiserver-etcd-client      Feb 08, 2022 10:25 UTC   363d            etcd-ca                 no      
apiserver-kubelet-client   Feb 08, 2022 10:25 UTC   363d            ca                      no      
controller-manager.conf    Feb 08, 2022 10:25 UTC   363d                                    no      
etcd-healthcheck-client    Feb 08, 2022 10:25 UTC   363d            etcd-ca                 no      
etcd-peer                  Feb 08, 2022 10:25 UTC   363d            etcd-ca                 no      
etcd-server                Feb 08, 2022 10:25 UTC   363d            etcd-ca                 no      
front-proxy-client         Feb 08, 2022 10:25 UTC   363d            front-proxy-ca          no      
scheduler.conf             Feb 08, 2022 10:25 UTC   363d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Feb 06, 2031 10:25 UTC   9y              no      
etcd-ca                 Feb 06, 2031 10:25 UTC   9y              no      
front-proxy-ca          Feb 06, 2031 10:25 UTC   9y              no    
```

## Dependency on Docker explained
- A container runtime is software that can execute the containers that make up a kubernetes pod
- Kubelet uses the container runtime interfaces as an abstraction so that you can use any compatible container runtime such as docker, ...
- The dockershim adapter allows the kubelet to interact with Docker as if Docker were a CRI compatible runtime
- References: https://kubernetes.io/docs/tasks/administer-cluster/migrating-from-dockershim/check-if-dockershim-deprecation-affects-you/#role-of-dockershim
