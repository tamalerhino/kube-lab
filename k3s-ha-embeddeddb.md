# Kubernetes using K3's High Avalability architecture using an Embedded Database (etcd)

###################################### DO NOT USE THIS FOR PROD ######################################
#################################### For educational purposes only ####################################

## Overview
The k3 documentation is not the greates for a beginer wanting to use an embedded db and HA which is as close to a prod deployment as youll get other than using an external DB.
Theres documentation for K3 HA architecture using an external DB and then theres documentation for a single server with an embedded DB. And because the HA needs a LB youll need the instructions for that too.
Then there are instructions for adding worker nodes which is confusing since you dont know if you are adding a server node  or a worker node since the instructions are so similar both using the same "node pool" terms. Then theres the issue that some instructions have you using the binary while others have you using a script.... So i decided to write my own from start to finish using just the script to make it easier.

At the end of this document you will have the following:
- A Load Balancer sending traffic to your HA Server Cluster.
- 3 Server nodes in an HA cluster using an embedded etcd database
- 3 Worker nodes in a cluster(rather node pool) connected to your Cluster of Server nodes
- 3 containers running nginx for a random little site i created :)

Because this is a lab i will follow best practices but again security is not top of mind.

## Assumptions
Because this is a lab im assuming you will be running this on some virtualization software/server like virtualbox/vmware workstation/ vmware player or ESXI/PROXMOX/KVM. However the instructions should still work if you decide to use different VM's,Bare Metal, or Raspberry Pi's

## Step 1. Setup your LB
Because kubernetes relies so hevily on knowing the ip addresses and/or hostnames its good to set this up first since youll need it for your "Register" or sometimes called "Fixed Registration Address" its just a fancy way of saying load balancer...

### To keep this doc small i moved these instructions to setup-lb.md


## Step 2. Setup your Distro
Because this is a lab i want to keep the footprint as small as possible because of this i will be using alpine which you can install on pretty much anything. However you are welcome to use ubuntu or any other distro that k3's support.

### I wanted to keep this as short as possible so i moved these instructions to their own file please see: setup-aline.md

## Step 3. Install K3S and initiallize the first node in the cluster.
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
Replace the follwing:

Variable | Description
---|---
`<cluster-token>`      | pre-generated secret token ex:`3ce58nF8xI5xajTWKHCD/0cj3fg6`
`<your-lb-ip-address>` | the ip of your load balancer ex `192.168.22.100`
...|...

## Step 3. Install K3S and add two new servers to the cluster.
Login to the second and third server and run the folowing comands.

```bash
curl -sfL https://get.k3s.io | sh -s - server \
--token=<cluster-token> \
--node-taint CriticalAddonsOnly=true:NoExecute --tls-san <your-lb-ip-address> \
--server https://<ip-of-the-first-server>:6443
```

See above for examples on what to replace.

Thats it you now have a HA Kubes cluster, meaning you can lose any 1 node and still be fine, if you lose 2 however everything will go down, follow the following node tolerance table to see how many more nodes you need to add to increase the number.

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
Alright now its time to add some Worker nodes! This is really where your containers will run - or at least this is where they should run however they can technecally run on the server nodes.

To do this login to each worker node and run the following commands:

```bash
curl -sfL https://get.k3s.io | sh -s - agent \
--server https://your-lb-ip-address:6443 \
--token <cluster-token>
```

Thats it! You now have a full HA kubes cluster with worker 3 worker nodes backed by an embedded database.

If you want to add any more sever nodes just do follow step 3 on any additional servers, and for workers follow step4 on any additional servers.

## Step 5. Download kubeconfig to local desktop
## TODO!! for now just git clone the repo to one of the server nodes and follow the next step


## Step 6. Deploy a thing!
Alright so now you got an HA cluster but nothing running on it..Lets change that!
In this repo you will find a directory called `kube-files` cd into it and then run:
```
kubectl apply -f chubbyunicorn-deploy.yaml
```
run the follwing to make sure they deployed
```
kubectl get pods
```


