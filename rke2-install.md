# rke2-install


```
vagrant status
```
```
loadbalancer1             not created (virtualbox)
loadbalancer2             not created (virtualbox)
kmaster1                  not created (virtualbox)
kmaster2                  not created (virtualbox)
kmaster3                  not created (virtualbox)
kworker1                  not created (virtualbox)
kworker2                  not created (virtualbox)
```
```
vagrant up kmaster1
vagrant up kmaster2
vagrant up kworker1
```


## ssh into nodes
```
vagrant ssh kmaster1
```
```
vagrant ssh kmaster2
```
```
vagrant ssh kworker1
```

## stop ufw or firewalld
```
sudo systemctl disable ufw
sudo systemctl stop ufw
```

# Server Node

## edit kmaster1 ```/etc/rancher/rke2/config.yaml```
```
sudo mkdir -p /etc/rancher/rke2/
sudo vim /etc/rancher/rke2/config.yaml
```
```
tls-san:
- your-k8s-dns.com
- 192.168.56.51
- 192.168.56.52
- 192.168.56.101
- 192.168.56.102
- 192.168.56.103
- 192.168.56.109
disable: rke2-ingress-nginx
cni:
- canal
node-ip: 192.168.56.101
```
## install rke2 server
```
curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_CHANNEL=v1.23  INSTALL_RKE2_TYPE="server" sh -
```



## start rke2-server
```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

## check rke2-server log
```
sudo journalctl -u rke2-server -f
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
kubectl get nodes -o wide -w
```
```
NAME       STATUS   ROLES                       AGE     VERSION          INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster1   Ready    control-plane,etcd,master   19m     v1.23.8+rke2r1   192.168.56.101   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
```
```
kubectl get pods -A -o wide -w
```

## show join node token
```
sudo cat /var/lib/rancher/rke2/server/node-token
```
```
K10400a2a885bd2afd9bbf90b3b7e3117f7b9b78393d240bc509699c04e111949d2::server:9b27c71333958cefa9c31ca3c4c3a674
```



# Install kube-vip to daemonsets

check INTERFACE
```
ip a
```
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:91:3c:79:38:5a brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 86388sec preferred_lft 86388sec
    inet6 fe80::91:3cff:fe79:385a/64 scope link
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:bb:42:85 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febb:4285/64 scope link
       valid_lft forever preferred_lft forever
```

## Set up environment
```
export VIP=192.168.56.109
export TAG=v0.4.4
export INTERFACE=enp0s8
export CONTAINER_RUNTIME_ENDPOINT=unix:///run/k3s/containerd/containerd.sock
export CONTAINERD_ADDRESS=/run/k3s/containerd/containerd.sock
```

## install
```
# Pull kube-vip RBAC manifest
curl -s https://kube-vip.io/manifests/rbac.yaml > /var/lib/rancher/rke2/server/manifests/kube-vip-rbac.yaml

# pull image
crictl pull docker.io/plndr/kube-vip:$TAG

# create alias
# on k3s `ctr` is a link to `k3s` which has the namespace set by default but on rke2 we
# have to specify the namespace

alias kube-vip="ctr --namespace k8s.io run --rm --net-host docker.io/plndr/kube-vip:$TAG vip /kube-vip"

# generate manifest
kube-vip manifest daemonset \
    --arp \
    --interface $INTERFACE \
    --address $VIP \
    --controlplane \
    --leaderElection \
    --taint \
    --services \
    --inCluster | tee /var/lib/rancher/rke2/server/manifests/kube-vip.yaml
```


## check kube-vip pods
```
kubectl get po -A | grep kube-vip
```
```
kube-vip-ds-8595m                           1/1     Running     0          48s
```
```
kubectl logs kube-vip-ds-8595m -n kube-system --tail 1
```
```
time="2021-03-12T12:29:38Z" level=info msg="Broadcasting ARP update for 192.168.56.109 (02:84:08:4a:dd:1c) via enp0s8"
```
# ping VIP
```
ping 192.168.56.109
```
```
PING 192.168.56.109 (192.168.56.109) 56(84) bytes of data.
64 bytes from 1192.168.56.109: icmp_seq=1 ttl=64 time=0.051 ms
64 bytes from 192.168.56.109: icmp_seq=2 ttl=64 time=0.043 ms
```
```
curl -k https://192.168.56.109:6443
```





## edit kmaster2 ```/etc/rancher/rke2/config.yaml```
```
sudo mkdir -p /etc/rancher/rke2/
sudo vim /etc/rancher/rke2/config.yaml
```
```
token: K10400a2a885bd2afd9bbf90b3b7e3117f7b9b78393d240bc509699c04e111949d2::server:9b27c71333958cefa9c31ca3c4c3a674
server: https://192.168.56.109:9345
tls-san:
- your-k8s-dns.com
- 192.168.56.51
- 192.168.56.52
- 192.168.56.101
- 192.168.56.102
- 192.168.56.103
- 192.168.56.109
disable: rke2-ingress-nginx
cni:
- canal
node-ip: 192.168.56.102
```

## install rke2 server
```
curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_CHANNEL=v1.23  INSTALL_RKE2_TYPE="server" sh -
```

## start rke2-server
```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```
```
kubectl get nodes -o wide
```
```
NAME       STATUS   ROLES                       AGE     VERSION          INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster1   Ready    control-plane,etcd,master   19m     v1.23.8+rke2r1   192.168.56.101   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
kmaster2   Ready    control-plane,etcd,master   10m     v1.23.8+rke2r1   192.168.56.102   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
```

# Agent Node

## edit kworker1 ```/etc/rancher/rke2/config.yaml```
```
sudo mkdir -p /etc/rancher/rke2/
sudo vim /etc/rancher/rke2/config.yaml
```
```
token: K10400a2a885bd2afd9bbf90b3b7e3117f7b9b78393d240bc509699c04e111949d2::server:9b27c71333958cefa9c31ca3c4c3a674
server: https://192.168.56.109:9345
node-ip: 192.168.56.201
```
## install rke2 agent
```
curl -sfL https://get.rke2.io | sudo INSTALL_RKE2_CHANNEL=v1.23  INSTALL_RKE2_TYPE="agent" sh -
```



## start rke2-agent
```
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```

## check rke2-agent log
```
sudo journalctl -u rke2-agent -f
```

```
kubectl get nodes -o wide
```
```
NAME       STATUS   ROLES                       AGE     VERSION          INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
kmaster1   Ready    control-plane,etcd,master   19m     v1.23.8+rke2r1   192.168.56.101   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
kmaster2   Ready    control-plane,etcd,master   10m     v1.23.8+rke2r1   192.168.56.102   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
kworker1   Ready    <none>                      3m48s   v1.23.8+rke2r1   192.168.56.201   <none>        Ubuntu 20.04.4 LTS   5.4.0-121-generic   containerd://1.5.13-k3s1
```

## test
```
kubectl create deploy nginx --image=nginx:stable
```