# Orion Role

## Description

The Orion role is responsible for setting up and managing the JunoFX Orion Cluster infrastructure. It handles the deployment and configuration of K3s clusters, including both CPU and GPU nodes, along with essential supporting applications and services.


## Requirements

### System Requirements
- Ubuntu 20.04 or later
- Minimum 4 CPU cores
- 8GB RAM minimum (16GB recommended for GPU nodes)
- For GPU nodes: NVIDIA GPU with compatible drivers

### Network Requirements
- All nodes must be able to communicate with each other
- Internet access for pulling container images
- Open ports:
  - 6443 (K3s API)
  - 10250 (Kubelet)

## Role Variables



### K3s Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `k3s_install_args` | See vars/main.yml | K3s installation arguments |
| `k3s_force_reinstall` | `false` | Force K3s reinstallation when restoring |
| `k3s_cluster_server_group_name` | `k3s_cluster_servers` | Control plane node group name |
| `k3s_kubeconfig_path` | `{{ playbook_dir }}/k3s_kubeconfig.yaml` | Path to store kubeconfig |

### Docker Registry Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_server` | `https://index.docker.io/v1/` | Docker registry server |
| `docker_registry_username` | `undefined` | Custom registry username |
| `docker_registry_password` | `undefined` | Custom registry password  |
| `docker_registry_secret_name` | `docker-registry-credentials` | K8s secret name for registry |

### Kubernetes Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `namespace` | `default` | Target K8s namespace |
| `kubeconfig_file` | `{{ k3s_kubeconfig_path }}` | Kubeconfig file location |

### ArgoCD Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `install_argocd_apps` | `[]` | List of ArgoCD applications to install |

### Deployment Control Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `deployment_scope` | Required | `cluster` or `apps` |
| `k3s_installation_target` | Required | `server` or `agent` |
| `node_type` | Required | `cpu` or `gpu` |
| `fully_restore_cluster` | Required | Whether to perform full restoration |

## K3s Installation Arguments

The role configures K3s with the following default arguments:
```yaml
k3s_install_args: |
  --disable servicelb 
  --disable traefik
  --kube-apiserver-arg enable-admission-plugins=PodNodeSelector
  --node-label junovfx/node=service
```


## Example Playbooks

### 1. Server Node Setup

```yaml
- hosts: k3s_cluster_servers
  roles:
    - role: orion
      vars:
        deployment_scope: cluster
        k3s_installation_target: server
        fully_restore_cluster: true
        k3s_install_args: "--disable servicelb --disable traefik"
```

### 2. GPU Agent Node Setup with Custom Registry

```yaml
- hosts: k3s_cluster_gpu_agents
  roles:
    - role: orion
      vars:
        deployment_scope: cluster
        k3s_installation_target: agent
        node_type: gpu
        fully_restore_cluster: true
        docker_registry: "registry.example.com"
```

### 3. Supporting Apps with ArgoCD Applications

```yaml
- hosts: localhost
  roles:
    - role: orion
      vars:
        deployment_scope: apps
        install_argocd: true
        install_argocd_apps:
          - name: monitoring
            namespace: monitoring
            path: charts/monitoring
```

## Handlers

The role includes handlers for:
- Restarting Docker service

## Templates
- ArgoCD Service

## Dependencies

- `community.docker` collection for Docker management
- `kubernetes.core` collection for K8s operations

## License

MIT

## Author Information

Last Updated: 2025-01-07

## Support

For issues and feature requests, please contact the JunoFX team.
