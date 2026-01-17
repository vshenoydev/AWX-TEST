# AWX-TEST

Ansible AWX playbooks for testing and demonstrating AWX automation on Minikube Kubernetes cluster.

## 📋 Overview

This repository contains Ansible playbooks designed to run on AWX (Ansible Tower) to manage and automate tasks on a Minikube Kubernetes cluster. The playbooks demonstrate various automation scenarios including system information gathering, package installation, Kubernetes cluster management, and resource backup.

## 🎯 Purpose

- **Learning AWX/Ansible Tower**: Hands-on examples for AWX configuration and job execution
- **Kubernetes Automation**: Demonstrate K8s cluster management using Ansible
- **CI/CD Integration**: Template for integrating Ansible automation with AWX
- **Best Practices**: Production-ready playbook structure and organization

## 📦 Repository Contents

### Playbooks

| Playbook | Description | Target |
|----------|-------------|--------|
| `minikube-test.yml` | Gather system information from minikube node | Minikube VM |
| `install-tools.yml` | Install useful CLI tools (htop, vim, curl, etc.) | Minikube VM |
| `k8s-info.yml` | Gather Kubernetes cluster information | localhost/K8s API |
| `backup-k8s.yml` | Backup Kubernetes resources to YAML files | localhost/K8s API |

### Additional Files

- `requirements.yml` - Ansible collection dependencies (kubernetes.core)

## 🚀 Quick Start

### Prerequisites

- **AWX** 24.6.1+ or Ansible Automation Platform running on Minikube
- **Minikube** cluster with kubectl configured
- **SSH access** to minikube node (for VM-targeted playbooks)
- **Kubernetes API access** (for K8s-targeted playbooks)

### AWX Setup

1. **Create Organization**
   ```
   Name: Demo Org
   ```

2. **Create Project**
   ```
   Name: AWX Test Project
   SCM Type: Git
   SCM URL: https://github.com/vshenoydev/AWX-TEST.git
   Update on Launch: ✓
   ```

3. **Create Inventory**
   
   **For Minikube VM playbooks:**
   ```yaml
   Name: Minikube Cluster
   Hosts:
     - minikube-node:
         ansible_host: 192.168.49.2  # Your minikube IP
         ansible_user: docker
         ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
   ```
   
   **For Kubernetes playbooks:**
   ```yaml
   Name: Localhost
   Hosts:
     - localhost:
         ansible_connection: local
         ansible_python_interpreter: /usr/bin/python3
   ```

4. **Create Credentials**
   ```
   Name: Minikube SSH Key
   Type: Machine
   Username: docker
   SSH Private Key: [Your minikube SSH key]
   Privilege Escalation: sudo
   ```

5. **Create Job Templates**

   | Template Name | Playbook | Inventory | Credential | Privilege Escalation |
   |---------------|----------|-----------|------------|---------------------|
   | 01 - Minikube System Info | minikube-test.yml | Minikube Cluster | Minikube SSH Key | ✓ |
   | 02 - Install Tools | install-tools.yml | Minikube Cluster | Minikube SSH Key | ✓ |
   | 03 - K8s Cluster Info | k8s-info.yml | Localhost | (none) | ✗ |
   | 04 - Backup K8s Resources | backup-k8s.yml | Localhost | (none) | ✗ |

## 📖 Playbook Details

### 1. minikube-test.yml

**Purpose:** Gather system information from the minikube node

**What it does:**
- Collects OS, kernel, memory, CPU information
- Checks disk usage
- Lists running Docker containers
- Creates a test file to verify write access

**Usage:**
```bash
# In AWX: Launch "01 - Minikube System Info" job template
```

**Requirements:**
- SSH access to minikube node
- Sudo privileges

---

### 2. install-tools.yml

**Purpose:** Install useful system utilities on minikube

**What it does:**
- Updates apt package cache
- Installs: htop, vim, curl, wget, net-tools, jq
- Verifies successful installation

**Usage:**
```bash
# In AWX: Launch "02 - Install Tools" job template
```

**Requirements:**
- SSH access to minikube node
- Sudo privileges
- Internet connectivity

**Note:** Uses apt package manager (Ubuntu/Debian). Modify for other distributions.

---

### 3. k8s-info.yml

**Purpose:** Gather comprehensive Kubernetes cluster information

**What it does:**
- Lists all namespaces
- Displays all pods across all namespaces
- Shows cluster endpoint information
- Gets node information

**Usage:**
```bash
# In AWX: Launch "03 - K8s Cluster Info" job template
```

**Requirements:**
- kubectl configured (runs on localhost/AWX container)
- kubernetes.core collection
- Access to Kubernetes API

**Collections Required:**
```yaml
collections:
  - kubernetes.core
```

---

### 4. backup-k8s.yml

**Purpose:** Backup Kubernetes resources to YAML files

**What it does:**
- Creates timestamped backup directory
- Exports deployments to YAML
- Exports services to YAML
- Exports configmaps to YAML
- Exports secrets to YAML
- Provides backup summary with file locations

**Usage:**
```bash
# In AWX: Launch "04 - Backup K8s Resources" job template
```

**Output:**
```
Backup Location: /tmp/k8s-backup-1704902400/
Files Created:
  - deployments.yaml
  - services.yaml
  - configmaps.yaml
  - secrets.yaml
```

**Requirements:**
- kubectl configured
- Write access to /tmp directory

---

## 🔧 Local Testing (Outside AWX)

You can test these playbooks locally before deploying to AWX:

### Setup

```bash
# Clone repository
git clone https://github.com/vshenoydev/AWX-TEST.git
cd AWX-TEST

# Install required collections
ansible-galaxy collection install -r requirements.yml

# Create inventory file
cat > inventory <<EOF
[minikube]
minikube-node ansible_host=192.168.49.2 ansible_user=docker

[localhost]
localhost ansible_connection=local
EOF
```

### Run Playbooks

```bash
# Test minikube playbooks
ansible-playbook -i inventory minikube-test.yml
ansible-playbook -i inventory install-tools.yml

# Test Kubernetes playbooks
ansible-playbook -i inventory k8s-info.yml
ansible-playbook -i inventory backup-k8s.yml
```

### Syntax Check

```bash
# Validate playbook syntax
ansible-playbook --syntax-check minikube-test.yml
ansible-playbook --syntax-check install-tools.yml
ansible-playbook --syntax-check k8s-info.yml
ansible-playbook --syntax-check backup-k8s.yml
```

---

## 🛠️ Troubleshooting

### Common Issues

#### 1. SSH Connection Failed

**Error:** `UNREACHABLE! => {"msg": "Failed to connect to the host via ssh"}`

**Solutions:**
- Verify minikube IP: `minikube ip`
- Test SSH manually: `ssh -i $(minikube ssh-key) docker@$(minikube ip)`
- Check AWX credential has correct SSH key
- Verify host variables in AWX inventory

---

#### 2. kubectl Command Not Found (k8s playbooks)

**Error:** `kubectl: command not found`

**Solutions:**
- Ensure playbooks target `localhost` (AWX container)
- Verify kubernetes.core collection is installed
- Check kubectl is available in execution environment

---

#### 3. Permission Denied / Sudo Errors

**Error:** `Missing sudo password`

**Solutions:**
- Enable "Privilege Escalation" in job template
- Configure passwordless sudo in minikube:
  ```bash
  minikube ssh
  echo "docker ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/docker
  ```

---

#### 4. Jinja2 Template Errors

**Error:** `template error while templating string: unexpected '.'`

**Solutions:**
- Use `shell` module instead of `command` for complex strings
- Escape Jinja2 braces: `{{ '{{' }}.Variable{{ '}}' }}`

---

## 📚 Additional Resources

### Documentation
- [AWX Documentation](https://ansible.readthedocs.io/projects/awx-operator/)
- [Ansible Documentation](https://docs.ansible.com/)
- [kubernetes.core Collection](https://docs.ansible.com/ansible/latest/collections/kubernetes/core/)

### Related Guides
- **AWX Complete Setup Guide** - Step-by-step AWX configuration
- **AWX Object Model Guide** - Understanding AWX architecture

### Minikube Resources
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-playbook`)
3. Commit your changes (`git commit -am 'Add new playbook'`)
4. Push to the branch (`git push origin feature/new-playbook`)
5. Create a Pull Request

### Contribution Guidelines

- Follow Ansible best practices
- Test playbooks locally before submitting
- Update README with new playbook documentation
- Use descriptive commit messages

---

## 📝 Playbook Standards

This repository follows these standards:

### Naming Conventions
```yaml
# Playbook names: lowercase with hyphens
playbook-name.yml

# Variables: snake_case
my_variable_name

# Tasks: Descriptive with proper capitalization
- name: Install required packages
```

### Structure
```yaml
---
- name: Descriptive playbook name
  hosts: target_group
  become: yes/no
  gather_facts: yes/no
  
  tasks:
    - name: Clear task description
      module_name:
        parameter: value
```

### Best Practices
- ✅ Always use explicit module parameters
- ✅ Include `changed_when` for command/shell tasks
- ✅ Use `register` for task output
- ✅ Add comments for complex logic
- ✅ Keep playbooks focused and modular
- ✅ Use handlers for service restarts
- ✅ Implement proper error handling

---

## 🔐 Security Considerations

### Credentials
- **Never commit credentials** to this repository
- Use AWX credential management for sensitive data
- Rotate SSH keys regularly
- Use different keys for dev/prod environments

### Secrets
- Store sensitive data in AWX credentials
- Use Ansible Vault for encrypted variables
- Never log sensitive information

### Access Control
- Use AWX RBAC for user permissions
- Limit inventory access to necessary users
- Audit job execution logs regularly

---

## 📊 Project Status

**Current Version:** 1.0.0  
**Status:** Active Development  
**Last Updated:** January 2026

### Changelog

#### Version 1.0.0 (January 2026)
- Initial release
- 4 core playbooks (system info, tools, K8s info, backup)
- AWX integration ready
- Complete documentation

### Roadmap

**Planned Features:**
- [ ] Advanced K8s deployment playbooks
- [ ] Monitoring and alerting automation
- [ ] Multi-cluster support
- [ ] Workflow templates
- [ ] Custom Ansible modules
- [ ] CI/CD pipeline integration

---

## 👤 Author

**Vivek Shenoy** ([@vshenoydev](https://github.com/vshenoydev))

---