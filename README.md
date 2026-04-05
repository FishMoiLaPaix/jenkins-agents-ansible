# Jenkins Agents Ansible

Ansible playbooks for provisioning and managing Jenkins agent machines.

## Managed Agents

| Agent | OS | Connection | Key Specs |
|-------|-----|-----------|-----------|
| **ubuntu-docker-agent-HP** | Ubuntu + Docker | SSH (192.168.0.40:2222) | 4 executors |
| **ubuntu-docker-agent-persoIA** | Ubuntu 24.04 + Docker 29.3 | SSH (192.168.0.46:22) | 62GB RAM, 1.8TB disk, 8 executors |
| **windows-docker-agent** | Windows + Docker Desktop | WinRM / JNLP inbound | Node.js 24, Python 3.13, Git 2.49 |

## Prerequisites

- **Ansible** >= 2.14 installed on the control machine
- **SSH keys** configured for Linux agents:
  - HP agent: key for `jenkins-agent` user on port 2222
  - persoIA agent: key at `~/.ssh/ai-workstation` for user `smerle`
- **WinRM** configured on the Windows agent (for Ansible connectivity)
- **Ansible collections**:
  ```bash
  ansible-galaxy collection install ansible.windows chocolatey.chocolatey community.general
  ```

## Usage

### Setup all agents

```bash
ansible-playbook playbooks/site.yml
```

### Setup Linux agents only

```bash
ansible-playbook playbooks/linux-agents.yml
```

### Setup Windows agent only

```bash
ansible-playbook playbooks/windows-agent.yml
```

### Verify all agents

```bash
ansible-playbook playbooks/verify.yml
```

### Target a specific agent

```bash
ansible-playbook playbooks/site.yml --limit ubuntu-docker-agent-HP
ansible-playbook playbooks/site.yml --limit ubuntu-docker-agent-persoIA
```

### Run specific roles with tags

```bash
# Install only Java
ansible-playbook playbooks/linux-agents.yml --tags java

# Install only Docker
ansible-playbook playbooks/linux-agents.yml --tags docker

# Setup only the Jenkins agent user and workspace
ansible-playbook playbooks/linux-agents.yml --tags jenkins-agent

# Run verification checks for a specific tool
ansible-playbook playbooks/verify.yml --tags docker
```

### Dry run (check mode)

```bash
ansible-playbook playbooks/site.yml --check --diff
```

## Available Tags

| Tag | Description |
|-----|-------------|
| `common` | Base packages and timezone |
| `java` | Java 21 (Temurin) installation |
| `docker` | Docker installation and service |
| `nodejs` | Node.js installation (Windows) |
| `python` | Python installation (Windows) |
| `git` | Git installation (Windows) |
| `jenkins-agent` | Agent user, workspace, SSH config |
| `verify` | Verification checks |
| `windows` | Windows-specific tasks |

## Project Structure

```
.
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   ├── hosts.yml               # Agent inventory
│   └── group_vars/
│       ├── linux.yml           # Linux agent variables
│       └── windows.yml         # Windows agent variables
├── roles/
│   ├── common/tasks/main.yml   # Base system setup
│   ├── java/tasks/main.yml     # Java 21 (Temurin)
│   ├── docker/tasks/main.yml   # Docker CE
│   ├── nodejs/tasks/main.yml   # Node.js, Python, Git (Windows)
│   └── jenkins-agent/tasks/    # Agent user and workspace
├── playbooks/
│   ├── site.yml                # All agents
│   ├── linux-agents.yml        # Linux agents
│   ├── windows-agent.yml       # Windows agent
│   └── verify.yml              # Verification checks
└── .gitignore
```

## Notes

- The **persoIA** agent connects as `smerle` with sudo, since Docker is already installed. The `jenkins-agent` user is created by Ansible.
- The **HP** agent connects directly as `jenkins-agent` on port 2222 (Docker container with SSH).
- The **Windows** agent uses JNLP inbound connection to Jenkins. WinRM is used only for Ansible provisioning. Update the IP/credentials in `inventory/hosts.yml` or use Ansible Vault.
- All playbooks run from the local network (LAN IPs). WAN hostnames are stored as variables for Jenkins configuration reference.
