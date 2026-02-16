# Kubernetes Ansible Deployment

This project uses Ansible to automate the deployment of a Kubernetes cluster on bare-metal servers. It sets up a single master node and multiple worker nodes, installs Calico for networking, and includes MetalLB for LoadBalancer services.

## Prerequisites

- Ansible 2.9+ installed on your control machine.
- One or more Ubuntu/Debian-based servers to act as Kubernetes nodes.
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

    - `kubernetes_version`: The version of Kubernetes to install. **Note:** The Ansible role currently hardcodes the repository to `v1.35`. To install a different version, you must update the paths in `roles/kubernetes/tasks/main.yml`.
    - `pod_network_cidr`: The CIDR for the pod network. The default (`192.168.0.0/16`) is used by `kubeadm init` and is compatible with the Calico CNI plugin installed by the `master` role.
    - `metallb_ip_range`: The IP address range for MetalLB to allocate for LoadBalancer services.

## Deployment

To deploy the Kubernetes cluster, run the main playbook from the root of the project:

```bash
ansible-playbook -i inventory/hosts.yml playbooks/site.yml
```

The playbook will execute the following steps in order:
1.  `common.yml`: Prepares all nodes with common configurations.
2.  `master.yml`: Initializes the Kubernetes master node.
3.  `workers.yml`: Joins the worker nodes to the cluster.
4.  `metallb.yml`: Deploys MetalLB for service load balancing.

## Playbooks and Roles Details

This project is organized into the following playbooks and roles:

### Playbooks

- `site.yml`: The main playbook that orchestrates the entire deployment.
- `common.yml`: Includes the `common`, `containerd`, and `kubernetes` roles to prepare all nodes.
- `master.yml`: Includes the `master` role to set up the Kubernetes master node.
- `workers.yml`: Includes the `worker` role to set up the Kubernetes worker nodes.
- `metallb.yml`: Includes the `metallb` role to deploy a load balancer.

### Roles

- `common`:
  - Disables swap memory.
  - Loads required kernel modules (`overlay`, `br_netfilter`).
  - Configures `sysctl` for Kubernetes networking.
  - Installs base packages like `curl` and `apt-transport-https`.

- `containerd`:
  - Installs `containerd` as the container runtime.
  - Generates a default configuration.
  - Configures `containerd` to use the `SystemdCgroup` driver, which is required by `kubelet`.

- `kubernetes`:
  - Adds the official Kubernetes `apt` repository.
  - Installs `kubelet`, `kubeadm`, and `kubectl`.
  - Enables the `kubelet` service.

- `master`:
  - Initializes the Kubernetes cluster using `kubeadm init`.
  - Sets up the `kubeconfig` file for the administrative user.
  - Deploys the [Calico](https://docs.projectcalico.org/) CNI (Container Network Interface) plugin.
  - Creates and saves a join command for worker nodes.

- `worker`:
  - Retrieves the join command from the master node.
  - Executes the command to join the worker to the Kubernetes cluster.

- `metallb`:
  - Deploys [MetalLB](https://metallb.universe.tf/) to the cluster using its official manifest.
  - Applies a configuration based on the `metallb_ip_range` variable defined in `group_vars/all.yml`.

## Project Structure

```
.
├── ansible.cfg
├── group_vars
│   └── all.yml
├── inventory
│   └── hosts.yml
├── playbooks
│   ├── common.yml
│   ├── master.yml
│   ├── metallb.yml
│   ├── site.yml
│   └── workers.yml
├── README.md
└── roles
    ├── common
    │   └── tasks
    │       └── main.yml
    ├── containerd
    │   └── tasks
    │       └── main.yml
    ├── kubernetes
    │   └── tasks
    │       └── main.yml
    ├── master
    │   └── tasks
    │       └── main.yml
    ├── metallb
    │   ├── tasks
    │   │   ├── main.yml
    │   │   └── templates
    │   │       └── metallb-config.yml.j2
    │   └── templates
    │       └── metallb-config.yml.j2
    └── worker
        └── tasks
            └── main.yml
```