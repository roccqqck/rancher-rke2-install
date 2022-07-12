# Rancher Install After install rke2
https://rancher.com/docs/rancher/v2.6/en/installation/install-rancher-on-k8s/

https://github.com/clemenko/rke_install_blog#automation


## Install Ingress Controller
The Rancher UI and API are exposed through an Ingress. 

For RKE2, and K3s installations, you don't have to install the Ingress controller manually because one is installed by default.

Ports 80 and 443 will be bound at every nodes IPs.

For others k8s distros, https://kubernetes.github.io/ingress-nginx/deploy/


## install helm and kubectl
https://helm.sh/docs/intro/install/
```
curl -#L https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
rke2: ```/var/lib/rancher/rke2/bin/kubectl```


## add repo
```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo add jetstack https://charts.jetstack.io
```

## helm install cert-manager
```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.8.2/cert-manager.crds.yaml

helm upgrade -i cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace
```

```
kubectl get pods --namespace cert-manager
```
```
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
```

## helm install rancher
```
helm upgrade -i rancher rancher-latest/rancher --create-namespace --namespace cattle-system --set hostname=rancher.dockr.life --set bootstrapPassword=bootStrapAllTheThings --set replicas=1
```
```--set bootstrapPassword```: rancher admin password ```bootStrapAllTheThings```

```--set hostname```: rancherurl ```https://rancher.dockr.life```


```
kubectl -n cattle-system get deploy rancher
```
```
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
```



```
helm list -A
```
```
NAME               	NAMESPACE    	REVISION	UPDATED                                	STATUS  	CHART                                       	APP VERSION
cert-manager       	cert-manager 	2       	2022-07-12 06:28:27.365548019 +0000 UTC	deployed	cert-manager-v1.8.2                         	v1.8.2
rancher            	cattle-system	2       	2022-07-12 06:28:38.049362018 +0000 UTC	failed  	rancher-2.6.6                               	v2.6.6
```

## set dns or ```/etc/hosts```
```
sudo vim /etc/hosts/
```
add WorkerNode-ip or LoadBalancer-ip
```
192.168.56.201  rancher.dockr.life
```

open https://rancher.dockr.life with browser