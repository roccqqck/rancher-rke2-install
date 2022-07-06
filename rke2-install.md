# rke2-install


```
vagrant up
```
```
vagrant ssh kubemaster
vagrant ssh kubenode01
```



# Server Node
```
curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=1.23  INSTALL_RKE2_TYPE="server" sudo sh -
```

## edit ```/etc/rancher/rke2/config.yaml```
```
sudo mkdir -p /etc/rancher/rke2/
sudo vim /etc/rancher/rke2/config.yaml
```
```
tls-san:
- your-k8s-dns.com
- 192.168.56.2
disable: rke2-ingress-nginx
cni:
- cilium
```

## start rke2-server
```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```


## copy kubeconfig file ```/etc/rancher/rke2/rke2.yaml``` to ```$HOME/.kube/config```
```
mkdir -p $HOME/.kube
sudo cp -i /etc/rancher/rke2/rke2.yaml $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


## export ```/var/lib/rancher/rke2/bin```

```
echo "export PATH=/var/lib/rancher/rke2/bin:$PATH" >> ~/.bashrc
source ~/.bashrc
```

```
kubectl get nodes -o wide
```

## show join node token
```
sudo cat /var/lib/rancher/rke2/server/node-token
```
```
K10400a2a885bd2afd9bbf90b3b7e3117f7b9b78393d240bc509699c04e111949d2::server:9b27c71333958cefa9c31ca3c4c3a674
```


# Agent Node
```
curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=1.23  INSTALL_RKE2_TYPE="agent" sudo sh -
```

## edit ```/etc/rancher/rke2/config.yaml```
```
sudo mkdir -p /etc/rancher/rke2/
sudo vim /etc/rancher/rke2/config.yaml
```
```
token: K10400a2a885bd2afd9bbf90b3b7e3117f7b9b78393d240bc509699c04e111949d2::server:9b27c71333958cefa9c31ca3c4c3a674
server: https://192.168.56.2:9345
```

## start rke2-agent
```
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```



## test
```
kubectl create deploy nginx --image=nginx:stable
```