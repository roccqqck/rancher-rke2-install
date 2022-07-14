# rancher-rke2-install

https://docs.rke2.io/install/quickstart/

https://docs.rancher.cn/docs/rke2/install/quickstart/_index/

https://gitlab.com/monachus/channel/-/tree/master/resources/2021-09-07-ha-rke2-kube-vip-rancher

https://kube-vip.io/docs/installation/daemonset/

https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/

https://rancher.com/docs/rancher/v2.6/en/installation/resources/k8s-tutorials/infrastructure-tutorials/nginx/

https://github.com/clemenko/rke_install_blog#automation





## Vagrant Environment
|Role|IP|OS|RAM|CPU|
|----|----|----|----|----|
|loadbalancer1|192.168.56.51|Ubuntu 20.04|1G|1|
|loadbalancer2|192.168.56.52|Ubuntu 20.04|1G|1|
|kmaster1|192.168.56.101|Ubuntu 20.04|2G|2|
|kmaster2|192.168.56.102|Ubuntu 20.04|2G|2|
|kmaster3|192.168.56.103|Ubuntu 20.04|2G|2|
|kworker1|192.168.56.201|Ubuntu 20.04|1G|1|
|kworker2|192.168.56.202|Ubuntu 20.04|1G|1|
|Virtual IP|192.168.56.50|Keepalived|-|-|
|Virtual IP|192.168.56.109|kube-vip / kube-apiserver 6443|-|-|
|Virtual IP|192.168.56.110|kube-vip / ingress 80 443|-|-|
|IP-RANGE|192.168.56.110|kube-vip Cloud Provider|-|-|


change nodeCount and cpu and memory at ```Vagrantfile```