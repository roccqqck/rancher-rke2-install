# Kube-VIP Cloud Provider Install for Service.Type.LoadBalancer
## master node
```
curl -sfL https://raw.githubusercontent.com/kube-vip/kube-vip-cloud-provider/main/manifest/kube-vip-cloud-controller.yaml > /var/lib/rancher/rke2/server/manifests/kube-vip-cloud-controller.yaml
```

set Service.Type.LoadBalancer ip range: 192.168.56.111-192.168.56.120 in ```/var/lib/rancher/rke2/server/manifests/kube-vip-config.yaml```
```
vim kube-vip-config.yaml
```
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubevip
  namespace: kube-system
data:
  range-global: 192.168.56.111-192.168.56.120
```

```
sudo cp kube-vip-config.yaml /var/lib/rancher/rke2/server/manifests
```

## test
```
kubectl create deploy nginx --image=nginx:stable
kubectl expose deploy nginx --port=80 --type=LoadBalancer
```
```
kubectl get services -A
```




# Config rke2-ingress-nginx service to Type.LoadBalancer


For RKE2, the Ingress controller is installed by default from helm.

Ports 80 and 443 will be bound at every Nodes IPs.

```
kubectl get helmchart -A
```
```
NAMESPACE     NAME                  AGE
kube-system   rke2-canal            55m
kube-system   rke2-coredns          55m
kube-system   rke2-ingress-nginx    55m
kube-system   rke2-metrics-server   55m
```


If you want to change the ```helm value```, use ```HelmChartConfig```

https://docs.rke2.io/networking/#nginx-ingress-controller

https://docs.rke2.io/helm/#customizing-packaged-components-with-helmchartconfig

```
kubectl get helmchartconfig -A
```
```
No resources found
```


```
vim rke2-ingress-nginx-config.yaml
```
```
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: rke2-ingress-nginx
  namespace: kube-system
spec:
  valuesContent: |-
    controller:
      service:
        enabled: true
        type: LoadBalancer
        loadBalancerIP: "192.168.56.110"
```
For more information, refer to the official nginx-ingress Helm Configuration Parameters.
https://github.com/kubernetes/ingress-nginx/blob/main/charts/ingress-nginx/values.yaml



```
sudo cp rke2-ingress-nginx-config.yaml /var/lib/rancher/rke2/server/manifests
```
```
kubectl get helmchartconfig -A
```
```
NAMESPACE     NAME                 AGE
kube-system   rke2-ingress-nginx   37m
```

```
kubectl get svc -A
```
```
NAMESPACE     NAME                                      TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
kube-system   rke2-ingress-nginx-controller             LoadBalancer   10.43.94.233   192.168.56.110   80:31476/TCP,443:31864/TCP   20m
kube-system   rke2-ingress-nginx-controller-admission   ClusterIP      10.43.2.131    <none>           443/TCP                      70m
```

now, we can access ingress from 192.168.56.110 

## set dns or ```/etc/hosts```
```
sudo vim /etc/hosts/
```
add Node-ip or LoadBalancer-ip
```
# LoadBalancer-ip
192.168.56.110  you.ingress1.host.com  
# Node-ip
192.168.56.101  you.ingress2.host.com
```
