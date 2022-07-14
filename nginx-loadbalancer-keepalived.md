
# nginx loadbalancer

```
vagrant up loadbalancer1
```
```
vagrant ssh loadbalancer1
```
## stop ufw or firewalld
```
sudo systemctl disable ufw
sudo systemctl stop ufw
```


## install nginx
https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/
```
sudo wget https://nginx.org/keys/nginx_signing.key
sudo apt-key add nginx_signing.key
```
```
sudo vi /etc/apt/sources.list
```
```
deb https://nginx.org/packages/ubuntu/ focal nginx
# deb-src https://nginx.org/packages/ubuntu/ focal nginx
```
```
sudo apt-get update
sudo apt-get install nginx
```
```
sudo systemctl enable nginx
sudo systemctl start nginx
```

## edit ```/etc/nginx/nginx.conf```
```
sudo vim /etc/nginx/nginx.conf
```
```
worker_processes 4;
worker_rlimit_nofile 40000;

events {
    worker_connections 8192;
}

stream {
    upstream rancher_servers_6443 {
        least_conn;
        server <IP_NODE_1>:6443 max_fails=3 fail_timeout=5s;
        server <IP_NODE_2>:6443 max_fails=3 fail_timeout=5s;
        server <IP_NODE_3>:6443 max_fails=3 fail_timeout=5s;
    }
    server {
        listen 6443;
        proxy_pass rancher_servers_6443;
    }

    upstream rancher_servers_9345 {
        least_conn;
        server <IP_NODE_1>:9345 max_fails=3 fail_timeout=5s;
        server <IP_NODE_2>:9345 max_fails=3 fail_timeout=5s;
        server <IP_NODE_3>:9345 max_fails=3 fail_timeout=5s;
    }
    server {
        listen     9345;
        proxy_pass rancher_servers_9345;
    }

}
```

```
sudo systemctl restart nginx
```