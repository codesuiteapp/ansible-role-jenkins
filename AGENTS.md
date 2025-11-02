# AGENTS.md - Ansible Role: Jenkins CI

Quick reference guide for AI agents working with this Ansible role.

## 1. Project Overview

**Type**: Ansible Role
**Purpose**: Automated deployment of Jenkins CI/CD server using Docker
**Primary Function**: Deploys and configures Jenkins container with Docker Compose
**Target Systems**: RHEL/CentOS, Debian/Ubuntu, Alpine, ArchLinux

**Key Features**:
- Docker-based Jenkins deployment
- Integration support: AWS, Kubernetes, ArgoCD, SSH, Flyway
- Automated configuration management
- Multi-platform support
- Traefik reverse proxy integration

## 2. Tech Stack

**Core Technologies**:
- Ansible (>= 2.1)
- Docker (>= 20.10)
- Docker Compose (v2.x)
- Python 3.6+
- Jinja2 templates

**Jenkins Stack**:
- Jenkins (containerized)
- Java runtime
- Docker-in-Docker support
- Maven integration

**Integration Tools**:
- AWS CLI
- Kubernetes (kubectl)
- ArgoCD CLI
- Traefik reverse proxy
- Flyway database migration

**Dependencies**:
- ansible.builtin modules
- community.docker collection
- Docker SDK for Python
- codesuiteapp.user role

## 3. Architecture & Project Structure

### Directory Layout

```
ansible-role-jenkins/
├── defaults/main.yml          # Default variables (image, ports, features)
├── vars/main.yml              # Internal variables (paths, Git config)
├── tasks/main.yml             # Main task flow
├── handlers/main.yml          # Service lifecycle handlers
├── templates/                 # Jinja2 templates
│   ├── docker-compose.yml.j2  # Container orchestration
│   ├── .env.j2                # Environment variables
│   ├── gitconfig.j2           # Git configuration
│   └── settings.xml.j2        # Maven settings
├── meta/main.yml              # Role metadata & dependencies
└── tests/test.yml             # Test playbook
```

### Architecture Flow

1. **Pre-requisites**: Docker installation (external requirement)
2. **User Setup**: Creates/configures admin user (via dependency role)
3. **Directory Creation**: Sets up required directory structure
4. **Integration Setup**: Configures AWS/K8s/ArgoCD/SSH if enabled
5. **Template Rendering**: Generates docker-compose.yml and configs
6. **Image Management**: Pulls/tags/pushes Docker images
7. **Container Deployment**: Launches Jenkins via Docker Compose

### Data Flow

```
Variables (defaults/vars)
  → Tasks (directory setup, file templating)
    → Templates (docker-compose.yml.j2, .env.j2)
      → Docker Compose
        → Jenkins Container
```

## 4. Core Domain Concepts

### Role Variables (Key Configurations)

**Required Variables**:
- `dkr_username`: Docker registry username
- `dkr_passwd`: Docker registry password
- `passwd_adm_user`: Admin user password

**Image Configuration**:
- `image_registry_server`: Primary registry (default: ghcr.io/codesuiteapp)
- `image_registry_server2`: Secondary registry (default: devops)
- `jenkins_image`: Image name (default: jenkins-docker)
- `jenkins_ver`: Image version (default: latest)

**Directory Structure**:
- `app_home_dir`: Application home (default: /app)
- `jenkins_home`: Jenkins config directory
- `jenkins_data`: Jenkins persistent data
- `nexus_data`: Maven repository

**Feature Flags** (defaults to false):
- `use_aws`: AWS CLI integration
- `use_kubeconfig`: Kubernetes config
- `use_argocd`: ArgoCD CLI
- `use_ssh`: SSH key management
- `use_flyway`: Database migration support
- `use_traefik`: Reverse proxy integration
- `use_dkr_net`: Custom Docker network

### Tasks Execution Order

1. Create base directories (app_home, jenkins_home, jenkins_data)
2. Create conditional directories (AWS, ArgoCD, SSH, Flyway)
3. Setup AWS credentials (if use_aws=true)
4. Setup ArgoCD config (if use_argocd=true)
5. Setup kubeconfig (if use_kubeconfig=true)
6. Setup SSH keys (if use_ssh=true)
7. Copy template files (.env, docker-compose.yml, gitconfig, settings.xml)
8. Install Docker SDK for Python
9. Login to Docker registry
10. Pull Jenkins image (if needed)
11. Tag and push image (if configured)
12. Trigger Jenkins restart handler

### Handlers

- `Start jenkins`: Launches container via docker-compose (with fallback)
- `Stop jenkins`: Stops container
- `Restart jenkins`: Restarts container (force-recreate)

All handlers use `docker_compose_v2` module with shell command fallback.

## 5. Coding Guidelines & Rules

### Must-Do

**Variable Management**:
- Always define required vars: `dkr_username`, `dkr_passwd`, `passwd_adm_user`
- Use `become: true` in playbooks for privilege escalation
- Set feature flags explicitly (use_aws, use_ssh, etc.)
- Use Ansible Vault for sensitive data

**File Permissions**:
- Directories: 0755 (standard), 0700 (AWS/SSH)
- Sensitive files: 0600 (credentials, keys)
- Config files: 0644
- Owner/Group: Use `adm_user` and `adm_group`

**Idempotency**:
- Tasks must be idempotent
- Use `state: directory` for mkdir operations
- Use `state: touch` with mode for file creation
- Register results and use conditionals

**Error Handling**:
- Use `ignore_errors: true` for optional tasks
- Provide fallback commands in handlers
- Check conditionals: `when: variable is defined`

### Don'ts

**Security**:
- DON'T commit credentials to version control
- DON'T use plain text passwords in playbooks
- DON'T skip file permission settings
- DON'T expose sensitive ports without firewall

**Configuration**:
- DON'T modify vars/main.yml for custom settings (use defaults override)
- DON'T hardcode paths (use variables)
- DON'T skip dependency installation
- DON'T run without Docker pre-installed

**Operations**:
- DON'T force push images without verification
- DON'T use `docker_type: ghcr.io` without proper authentication
- DON'T enable all features without resource planning
- DON'T ignore handler failures (check logs)

### Template Best Practices

**Jinja2 Templates**:
- Use `{% if condition %}` for conditional blocks
- Use `{{ variable }}` for substitutions
- Use `{% for item in list %}` for iterations
- Test with different variable combinations

**Docker Compose Template**:
- Always validate YAML syntax
- Use environment variables from .env.j2
- Mount volumes conditionally based on feature flags
- Include restart policy: `unless-stopped`

## 6. Configuration & Environment

### Environment Variables (in .env.j2)

```
DOCKER_IMAGE={{ image_registry_server2 }}/{{ jenkins_image }}:{{ jenkins_ver }}
jenkins_port={{ jenkins_port }}
trf_domain={{ traefik_domain }}  # if use_traefik
```

### Volume Mounts

**Always Mounted**:
- `{{ jenkins_data }}:/var/jenkins_home` - Jenkins home
- `{{ nexus_data }}:{{ nexus_data }}` - Maven repository
- `{{ jenkins_home }}/gitconfig:/root/.gitconfig` - Git config
- `{{ jenkins_home }}/settings.xml:/root/.m2/settings.xml` - Maven settings
- `/var/run/docker.sock:/var/run/docker.sock` - Docker socket
- `/bin/docker:/bin/docker` - Docker binary

**Conditional Mounts**:
- AWS: `{{ jenkins_home }}/.aws:/root/.aws`
- Kubernetes: `{{ jenkins_home }}/kubeconfig:/root/.kube/config`
- ArgoCD: `{{ jenkins_home }}/argocd/config:/root/.config/argocd/config`
- SSH: `{{ jenkins_home }}/.ssh/id_rsa:/root/.ssh/id_rsa`
- Flyway: `/data:/data`

### Port Mappings

- `{{ jenkins_port }}:8080` - Jenkins web UI
- `9090:9090` - Additional service port
- `50000:50000` - Jenkins agent port

### Network Configuration

**Standalone Mode** (use_dkr_net=false):
- No custom network
- Host networking for inter-container communication

**Network Mode** (use_dkr_net=true):
- Network: `{{ dkr_network }}` (external: true)
- Must pre-create network: `docker network create {{ dkr_network }}`

**Extra Hosts** (use_extra_hosts=true):
- Format: `hostname:ip_address`
- Example: `extra_hosts: ["gitlab.local:192.168.1.10"]`

### Typical Deployment Scenarios

**Development**:
```yaml
env_profile: dev
jenkins_port: 9876
use_traefik: false
use_dkr_net: false
```

**Production**:
```yaml
env_profile: prod
jenkins_port: 8080
use_traefik: true
use_traefik_tls: true
use_dkr_net: true
use_aws: true
use_kubeconfig: true
```

### Execution

**Install Role**:
```bash
ansible-galaxy install codesuiteapp.jenkins
```

**Run Playbook**:
```bash
ansible-playbook -i inventory playbook.yml --ask-become-pass
```

**With Vault**:
```bash
ansible-playbook -i inventory playbook.yml --ask-vault-pass
```

### Dependencies

**External Role**:
- Name: `codesuiteapp.user`
- Source: https://github.com/codesuiteapp/ansible-role-user.git
- Purpose: User/group management
- Auto-installed: Yes (via meta/main.yml)

**Python Packages**:
- docker (installed via pip in tasks)

**System Requirements**:
- Docker daemon running
- docker-compose binary available
- Sufficient disk space for Jenkins data

---

**Last Updated**: 2025-11-02
**Version**: Latest (main branch)
