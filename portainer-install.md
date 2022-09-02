# k8s install portainer

https://docs.portainer.io/start/install/server/kubernetes/baremetal

First add the Portainer Helm repository by running the following commands:
```
helm repo add portainer https://portainer.github.io/k8s/
helm repo update
```


### helm install
```
helm install --create-namespace -n portainer portainer portainer/portainer \
    --set service.type=ClusterIP \
    --set ingress.enabled=true \
    --set ingress.hosts[0].host=portainer.example.io \
    --set ingress.hosts[0].paths[0].path="/"
```


if it is not working
```
helm install --create-namespace -n portainer portainer portainer/portainer \
    --set service.type=LoadBalancer \
```

edit ```portainer-ingress.yml```
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: portainer
  namespace: portainer
spec:
  rules:
  - host: portainer-ingress.info
    http:
      paths:
      - backend:
          service:
            name: portainer
            port:
              number: 9000
        path: /
        pathType: Prefix
```

apply ```portainer-ingress.yml```
```
kubectl apply -f portainer-ingress.yml -n portainer
```


http://portainer-ingress.info