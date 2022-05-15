# NGINX Load Balancer
For our K3s lab we will be deploying a very very...very simple Nginx Load Balancer.
Again assuming here you are using alpine but just change `apk add` to `apt install` for Debian/Ubuntu or `yum install` for RHEL.

## Step 1. Update your package manager and install Nginx
setup a static IP address by editing your `/etc/network/interfaces` file
Then update your packages and install Nginx(see instructions in setup-alpine.md)
```
apk update && apk upgrade
```
```
apk add curl vim nginx nginx-mod-stream
```

## Step 2. Configure NGINX
Edit the `/etc/nginx/nginx.conf` file so it has only the following content REPLACING THE IP ADDRESSES TO YOUR KB-SERVER-{1-3} IP ADDRESSES:
```bash
load_module /usr/lib/nginx/modules/ngx_stream_module.so;

events {}

stream {
  upstream k3s_servers {
    server 192.168.22.101:6443;
    server 192.168.22.102:6443;
    server 192.168.22.103:6443;
  }

  server {
    listen 6443;
    proxy_pass k3s_servers;
  }
}
```
Run this command so nginx automatically starts
```bash
rc-update add nginx default
```
Once that is done reboot

