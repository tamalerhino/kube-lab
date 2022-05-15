# Alpine

For our Kubernetes lab we will be using the Alpine Distro since it is a tiny Distro and doesn't take up too much to run.

YOU WILL NEED TO MAKE 7 of these!!
- 1 NGINX LB
- 3 Node (master servers)
- 3 Worker nodes

## Download Alpine
Download Alpine from  "https://www.alpinelinux.org/downloads/"
Download the "Standard" version for your architecture.

## Install Alpine
Once it is downloaded, load the ISO, or burn it to a flash drive using etcher "https://www.balena.io/etcher/"

Boot from the ISO or USB.
For the username use `root` - no password is needed
Then run:
```
setup-alpine
```
Follow the instructions but this should get you through
- `us`
- `us` (its going to ask for an alternative keyboard)
- `kb-server-1` Replace `server` with `worker` for a worker node and `1` to the number you want. Also for the Nginx LB name it `nginx-lb`

Many of these i just hit `enter` on for the defaults
- `eht0`
- `dhcp` Using DHCP for now just to get it to work we will set a static address later
- `no`
- `<root_password>`
- `<confirm_root_password>`
- `UTC` or whatever timezone you're in
- `none`
- `chrony`
- `1`
- `openssh`
- `sda` whatever disk you want to use
- `sys` or lvm
- `y`
- `reboot`

## Configure Alpine

Update and upgrade alpine and install curl
```
apk update && apk upgrade
```
```
apk add curl vim
```

Since were running Alpine you will need to do some additional preparation, from the [official K3s docs.](https://rancher.com/docs/k3s/latest/en/advanced/#additional-preparation-for-alpine-linux-setup)

Update the `/etc/update-extlinux.conf` file by adding(you will add it to what is already there for example mine says `default_kernel_opts="pax_nouderef quiet rootfstype=ext4` so i just add the `cgroup` etc to that line.):
```bash
default_kernel_opts="...  cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory"
```

then run

```bash
update-extlinux
```

IP GUIDE
Server | IP Address
---|---
NGINX-LB | 192.68.22.100
KB-Server-1 | 192.68.22.101
KB-Server-2| 192.68.22.102
KB-Server-3| 192.68.22.103
KB-Worker-1| 192.68.22.111
KB-Worker-2| 192.68.22.112
KB-worker-3| 192.68.22.113
...|...


Lets go ahead and set a static address, again replacing it with your configuration for each server
open `/etc/network/interfaces` and replace the line that says `iface eth0 inet dhcp` to:
```bash
iface eth0 inet static
        address 192.168.22.100
       netmask 255.255.255.0
        gateway 192.168.22.2
```

Finally reboot

```bash
reboot
```
That's it!


## Additional Configuration

### SSHD
It might be a pain to have to write everything through the console, alpine has sshd installed and running but you cant login with the root user unless you make some changes

Edit the `/etc/ssh/sshd_config` file by:

Uncommenting the line
```
#PermitRootLogin prohibit-password
```
and change `prohibit-password` to `yes` as shown below
```
PermitRootLogin yes
```
then restart sshd
```
service sshd restart
```
now you should be able to ssh in.
