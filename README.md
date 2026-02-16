# Kubernetes Ansible Deployment

This project uses Ansible to automate the deployment of a Kubernetes cluster on bare metal servers. It sets up a single master node and multiple worker nodes, and includes MetalLB for load balancing.

## Prerequisites

- Ansible 2.9+ installed on your control machine.
- One or more servers (virtual or physical) to act as Kubernetes nodes.
- SSH access to the nodes from the control machine, with a user that has passwordless sudo privileges.

## Configuration

1.  **Inventory:**
    Update the `inventory/hosts.yml` file to match your server inventory. Replace the example IP addresses with the actual IPs of your master and worker nodes.

    ```yaml
    all:
      children:
        masters:
          hosts:
            master:
              ansible_host: <MASTER_IP>
        workers:
          hosts:
            worker1:
              ansible_host: <WORKER1_IP>
            worker2:
              ansible_host: <WORKER2_IP>
    ```

2.  **Variables:**
    Modify the variables in `group_vars/all.yml` to suit your environment:

    - `kubernetes_version`: The version of Kubernetes to install.
    - `pod_network_cidr`: The CIDR for the pod network. The default is `192.168.0.0/16`. This is compatible with Calico.
    - `metallb_ip_range`: The IP address range for MetalLB to use for LoadBalancer services.

## Deployment

To deploy the Kubernetes cluster, run the main playbook:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/site.yml
```

The playbook will:
1.  Run common configurations on all nodes.
2.  Set up the Kubernetes master node.
3.  Set up the Kubernetes worker nodes and join them to the cluster.
4.  Install and configure MetalLB.

## Playbooks and Roles

This project is organized into the following playbooks and roles:

### Playbooks

- `site.yml`: The main playbook that orchestrates the entire deployment.
- `common.yml`: Installs common packages and configures all nodes.
- `master.yml`: Sets up the Kubernetes master node.
- `workers.yml`: Sets up the Kubernetes worker nodes.
- `metallb.yml`: Deploys MetalLB for service load balancing.

### Roles

- `common`: Common tasks for all nodes, including package installation and system configuration.
- `containerd`: Installs and configures the containerd container runtime.
- `kubernetes`: Installs `kubeadm`, `kubelet`, and `kubectl`.
- `master`: Initializes the Kubernetes cluster on the master node.
- `worker`: Joins the worker nodes to the cluster.
- `metallb`: Deploys MetalLB to the cluster.