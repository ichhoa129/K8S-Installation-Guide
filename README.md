# K8S INSTALLATION GUIDE
These are the steps to create a high-availability K8S Cluster with 3 Master nodes, including Monitoring tools like Rancher and Grafana.

## ENVIRONMENT PREPARATION
Virtual Machines:
1. Master Nodes:
- Description: To install k8s Control-plane
- amount: 3/5/7
- OS: Ubuntu
- CPU: 4
- RAM: 8 GB
- Disk: 100GB

2. Worker Nodes:
- Description: To install applications
- amount: > 1
- OS: Ubuntu
- CPU: 4 (Based on the application requirement)
- RAM: 8 GB (Based on the application requirement)
- Disk: 40GB (Based on the application requirement)

3. HA Proxy Load balancer:
- Description: Load balancer for 3 master nodes
- amount: 1
- OS: Ubuntu
- CPU: 2
- RAM: 4 GB
- Disk: 40 GB

4. Rancher Server:
- Description: Install Rancher to monitor clusters
- amount: 1
- OS: Ubuntu
- CPU: 1
- RAM: 4 GB
- Disk: 40 GB

## Steps 
1. [Initialize Cluster](1.CLUSTER.md)
2. [Install Ingress](2.INGRESS.md)
3. [Install Rancher](3.RANCHER.md)
4. [Install Grafana](4.GRAFANA.md)
