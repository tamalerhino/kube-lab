# Kubernetes using K3s High Availability architecture using an Embedded Database (etcd)

###################################### DO NOT USE THIS FOR PROD ######################################
#################################### For educational purposes only ####################################

## Overview
The K3s documentation is not the greatest for a beginner wanting to use an embedded db and HA which is as close to a prod deployment as you'll get other than using an external DB.
There's documentation for K3s HA architecture using an external DB and then there's documentation for a single server with an embedded DB. And because the HA needs a LB you'll need the instructions for that too.
Then there are instructions for adding worker nodes which is confusing since you don't know if you are adding a server node  or a worker node since the instructions are so similar both using the same "node pool" terms. Then there's the issue that some instructions have you using the binary while others have you using a script.... So i decided to write my own from start to finish using just the script to make it easier.

At the end of this document you will have the following:
- A Load Balancer sending traffic to your HA Server Cluster.
- 3 Server nodes in an HA cluster using an embedded etcd database
- 3 Worker nodes in a cluster(rather node pool) connected to your Cluster of Server nodes
- 3 Containers running Nginx for a random little site i created :)

Because this is a lab I will follow best practices but again security is not top of mind.

## Assumptions
Because this is a lab I'm assuming you will be running this on some virtualization software/server like VirtualBox/VMware workstation/ VMware player or ESXI/Proxmox/KVM. However the instructions should still work if you decide to use different VM's,Bare Metal, or Raspberry Pi's

## Step 1. Setup your LB
Because Kubernetes relies so heavily on knowing the IP addresses and/or Hostname's its good to set this up first since you'll need it for your "Register" or sometimes called "Fixed Registration Address" its just a fancy way of saying Load Balancer...

### To keep this doc small i moved these instructions to setup-lb.md


## Step 2. Setup your Distro
Because this is a lab i want to keep the footprint as small as possible because of this i will be using alpine which you can install on pretty much anything. However you are welcome to use Ubuntu or any other Distro that K3s support.

### I wanted to keep this as short as possible so i moved these instructions to their own file please see: setup-alpine.md

## Step 3. Install K3s and initialize the first node in the cluster.
On the first server in my case `kb-server-1` run the following commands.

First generate a cluster-token, this is a secret so treat it as one!

```bash
openssl rand -base64 21 > cluster-token
```

```bash
curl -sfL https://get.k3s.io | sh -s - server \
--token=<cluster-token> \
--node-taint CriticalAddonsOnly=true:NoExecute --tls-san <your-lb-ip-address> \
--cluster-init
```
Replace the following:

Variable | Description
---|---
`<cluster-token>`      | pre-generated secret token ex:`3ce58nF8xI5xajTWKHCD/0cj3fg6`
`<your-lb-ip-address>` | the IP of your Load Balancer ex `192.168.22.100`
...|...

## Step 3. Install K3s and add two new servers to the cluster.
Login to the second and third server and run the following commands.

```bash
curl -sfL https://get.k3s.io | sh -s - server \
--token=<cluster-token> \
--node-taint CriticalAddonsOnly=true:NoExecute --tls-san <your-lb-ip-address> \
--server https://<ip-of-the-first-server>:6443
```

See above for examples on what to replace.

That's it you now have a HA Kubes cluster, meaning you can lose any 1 node and still be fine, if you lose 2 however everything will go down, follow the following node tolerance table to see how many more nodes you need to add to increase the number.

Total Number of nodes | Failed Node Tolerance
---|---
1|0
2|0
3|1
4|1
5|2
6|2
...|...


### Test that it worked by running the following command on any node
```
k3s kubectl get nodes
```


## Step 4. Install K3s and add Worker Nodes.
Alright now its time to add some Worker nodes! This is really where your containers will run - or at least this is where they should run however they can technically run on the server nodes.

To do this login to each worker node and run the following commands:

```bash
curl -sfL https://get.k3s.io | sh -s - agent \
--server https://your-lb-ip-address:6443 \
--token <cluster-token>
```

That's it! You now have a full HA Kubes cluster with worker 3 worker nodes backed by an embedded database.

If you want to add any more sever nodes just do follow step 3 on any additional servers, and for workers follow step 4 on any additional servers.

## Step 5. Download kubeconfig to local desktop
The easiest way (not secure!) is to login to one of the Servers ex: `kb-server-1`
and cat out the contents of the K3s.yml to copy to a kubeconfig on your local dev workstation.
```bash
cat /etc/rancher/k3s/k3s.yaml
```
ON WORKSTATION:
Install kubectl following this [link](https://kubernetes.io/docs/tasks/tools/)
if kubeconfig file does not exist create it using
```bash
touch ~/.kube/config
```
And copy the contents from the `k3s.yml` into `config` file
Make sure you replace the line where it says `server:` to the IP of your lb in my case `server: https://192.168.22.100:6443`

## OPTIONAL
Optionally you can install the Kubernetes dashboard, its not needed but its node to have Web UI especially when you are first learning.
For the most up to date instructions click [here](https://github.com/kubernetes/dashboard)
To install run
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml
```
Create a Dashboard Admin user

If you are going to login you will need to create a Admin user for the dashboard and the role to give you permissions to the dashboard.

In kube-files you will find the dashboard.admin-user.yml and dashboard.admin-user-role.yml files ready to go. You can change the "admin-user" to a different name if you want, otherwise just leave it all as is. Likewise you can create more users and add them to the dashboard admin role.

Run the following to create your user.
```
kubectl create -f dashboard.admin-user.yml -f dashboard.admin-user-role.yml
```
Get your admin user token(bearer token), since the dashboard will ask for it.
```
kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token
```
Then to make it available locally run
```
kubectl proxy
```
now it should be available at the following address
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```
Once you login it will ask for a bearer token, use the one we got with our grep command.

## Step 6. Deploy a thing!
Alright so now you got an HA cluster but nothing running on it..Lets change that!
In this repo you will find a directory called `kube-files` cd into it and then run:
```
kubectl apply -f chubbyunicorn-deploy.yaml
```
run the following to make sure they deployed
```
kubectl get pods
```

