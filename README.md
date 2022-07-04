# kube-lab
Instructions for a Kubernetes lab

The idea is to create a k3s cluster with 3 control and 3 worker nodes, all behind a simple nginx load balancer for HA.

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
