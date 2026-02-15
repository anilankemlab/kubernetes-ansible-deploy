# Kubernetes Ansible Deployment (Bare Metal / Proxmox / Homelab)

This repository deploys a production-ready Kubernetes cluster using Ansible.

Cluster layout:

master   192.168.1.20
worker1  192.168.1.21
worker2  192.168.1.22

Features:

- containerd runtime
- kubeadm cluster setup
- Calico CNI networking
- MetalLB LoadBalancer support
- Fully automated worker join
- Idempotent Ansible roles
- Ready for GitHub Actions runner

Requirements:

- Ubuntu 22.04 / 24.04
- Ansible installed on runner
- Root SSH access to all nodes
- Passwordless SSH configured

Test SSH:

ssh root@192.168.1.20
ssh root@192.168.1.21
ssh root@192.168.1.22

Deploy cluster:

ansible-playbook playbooks/site.yml

Verify cluster:

ssh root@192.168.1.20

kubectl get nodes

Verify MetalLB:

kubectl get svc

Test LoadBalancer:

kubectl create deployment nginx --image=nginx

kubectl expose deployment nginx \
--port=80 \
--type=LoadBalancer

Access:

http://192.168.1.240


Repository Structure:

inventory/       Hosts
group_vars/      Global variables
roles/           Modular roles
playbooks/       Execution playbooks


MetalLB range:

192.168.1.240-192.168.1.250

Change in:

group_vars/all.yml


Future additions:

- Ingress Controller
- Helm
- Monitoring
- CI/CD automation
