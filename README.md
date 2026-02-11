# Mercan - Modular Ansible Honeypot Deployment Framework

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

A production-ready, highly modular Ansible framework for deploying multiple honeypot systems using Docker Compose. Designed with extensibility in mind, allowing easy addition of new honeypots following a well-established pattern.

## 🎯 Overview

Mercan provides a complete infrastructure for deploying and managing honeypot systems across multiple Linux distributions. The architecture is built around the principle of **modularity** - each honeypot is a self-contained role following identical conventions, making it trivial to add new honeypots to the framework.

### Key Features

- **🔧 Highly Modular Architecture**: Each honeypot is an independent role following the same pattern
- **🐧 Distribution Agnostic**: Works across Ubuntu, Debian, CentOS, Rocky Linux, RHEL, Fedora, AlmaLinux
- **🐋 Docker-Based**: All honeypots run in isolated Docker containers with Docker Compose
- **🔄 Dynamic Registry System**: Add/remove honeypots by editing configuration, not code
- **⚙️ Systemd Integration**: Auto-start on boot with proper service management
- **📊 Centralized Logging**: Structured logging with automatic log rotation
- **🔥 Firewall Management**: Automatic firewall configuration (iptables/firewalld/ufw)
- **🔒 SELinux Support**: Proper SELinux context handling for RHEL-based systems
- **💾 Backup System**: Automated timestamped backups with retention policies
- **📈 Health Monitoring**: Built-in health checks and status reporting

## 📋 Prerequisites

- **Ansible**: Version 2.10 or higher
- **Target Systems**: 
  - Ubuntu 18.04+, Debian 10+
  - CentOS 7+, Rocky Linux 8+, AlmaLinux 8+, RHEL 7+, Fedora 35+
  - Minimum 2GB RAM, 20GB disk space
  - Python 3.6+
- **Network**: Internet access for Docker image pulls
- **Privileges**: Root or sudo access on target systems

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/oncumuhammed/mercan.git
cd mercan
```

### 2. Configure Inventory

Edit `inventory/hosts.yml` with your target servers:

```yaml
all:
  children:
    honeypot_servers:
      hosts:
        honeypot-01:
          ansible_host: 192.168.1.100
          ansible_user: root
```

### 3. Customize Configuration

Edit `group_vars/all.yml` to enable/disable honeypots and configure settings:

```yaml
honeypots:
  cowrie:
    enabled: true  # Set to false to disable
    ports:
      - { host: 2222, container: 2222, protocol: "tcp" }
```

### 4. Deploy Honeypots

```bash
# Deploy all enabled honeypots
ansible-playbook playbooks/deploy.yml

# Deploy specific honeypot(s)
ansible-playbook playbooks/deploy.yml --tags cowrie,dionaea

# Dry run (check mode)
ansible-playbook playbooks/deploy.yml --check
```

### 5. Verify Deployment

```bash
# Check status of all honeypots
ansible-playbook playbooks/status.yml

# Check logs
ssh your-server "tail -f /opt/cowrie/logs/*.log"
```

## 🍯 Included Honeypots

| Honeypot | Purpose | Ports | Status |
|----------|---------|-------|--------|
| **Cowrie** | SSH/Telnet honeypot | 2222 (SSH), 2223 (Telnet) | ✅ Included |
| **Dionaea** | Multi-protocol honeypot | 21, 80, 443, 445, 1433, 3306, 5060, 11211 | ✅ Included |
| **Honeytrap** | Advanced trap framework | 8000-8010 | ✅ Included |
| **Conpot** | ICS/SCADA honeypot | 8800, 102, 502, 161/UDP, 47808 | ✅ Included |
| **Elasticpot** | Elasticsearch honeypot | 9200 | ✅ Included |

## 📖 Playbook Usage

### Deploy Playbook

Deploys infrastructure (Docker, common packages) and all enabled honeypots.

```bash
# Full deployment
ansible-playbook playbooks/deploy.yml

# Deploy specific honeypot
ansible-playbook playbooks/deploy.yml --tags cowrie

# Skip infrastructure setup (if already installed)
ansible-playbook playbooks/deploy.yml --skip-tags infrastructure
```

### Update Playbook

Updates honeypots by pulling latest Docker images and recreating containers.

```bash
# Update all honeypots
ansible-playbook playbooks/update.yml

# Update specific honeypot
ansible-playbook playbooks/update.yml --tags dionaea
```

### Backup Playbook

Creates timestamped tar.gz backups of data and configuration.

```bash
# Backup all honeypots
ansible-playbook playbooks/backup.yml

# Backup specific honeypot
ansible-playbook playbooks/backup.yml --tags cowrie
```

Backups are stored in `/var/backups/honeypots/` by default. To restore:

```bash
tar -xzf /var/backups/honeypots/cowrie_20260211T120000.tar.gz -C /
```

### Uninstall Playbook

Removes honeypots and optionally their data.

```bash
# Uninstall but keep data
ansible-playbook playbooks/uninstall.yml

# Uninstall and remove data
ansible-playbook playbooks/uninstall.yml --extra-vars "remove_data=true"

# Skip confirmation prompt
ansible-playbook playbooks/uninstall.yml --extra-vars "auto_approve=true"
```

### Status Playbook

Checks health and status of all honeypots.

```bash
# Check all honeypots
ansible-playbook playbooks/status.yml

# Check specific honeypot
ansible-playbook playbooks/status.yml --tags elasticpot
```

## 🏗️ Architecture

### Directory Structure

```
mercan/
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   ├── hosts.yml              # Active inventory
│   └── hosts.example.yml      # Example inventory
├── group_vars/
│   └── all.yml                # Global variables & honeypot registry
├── playbooks/
│   ├── deploy.yml             # Full deployment
│   ├── update.yml             # Update containers
│   ├── backup.yml             # Backup data
│   ├── uninstall.yml          # Remove honeypots
│   └── status.yml             # Health check
└── roles/
    ├── common/                # OS prep, packages, firewall
    ├── docker/                # Docker + Compose installation
    ├── honeypot_base/         # Shared tasks for all honeypots
    │   ├── tasks/
    │   │   ├── deploy.yml
    │   │   ├── backup.yml
    │   │   ├── uninstall.yml
    │   │   └── status.yml
    │   └── templates/
    │       ├── systemd.service.j2
    │       └── logrotate.j2
    ├── cowrie/                # Cowrie honeypot
    │   ├── tasks/main.yml
    │   ├── templates/docker-compose.yml.j2
    │   ├── defaults/main.yml
    │   └── meta/main.yml
    ├── dionaea/               # Dionaea honeypot (same structure)
    ├── honeytrap/             # Honeytrap honeypot (same structure)
    ├── conpot/                # Conpot honeypot (same structure)
    └── elasticpot/            # Elasticpot honeypot (same structure)
```

### Data Directory Structure

Each honeypot stores data under `/opt/<honeypot-name>/`:

```
/opt/
├── cowrie/
│   ├── data/              # Persistent data
│   ├── logs/              # Log files
│   ├── config/            # Configuration files
│   └── docker-compose.yml
├── dionaea/
│   ├── data/
│   ├── logs/
│   ├── config/
│   └── docker-compose.yml
└── ...
```

## ➕ Adding a New Honeypot

The modular architecture makes it easy to add new honeypots. Follow this pattern:

### Step 1: Create Role Directory

```bash
mkdir -p roles/new_honeypot/{tasks,templates,defaults,meta}
```

### Step 2: Create defaults/main.yml

```yaml
---
honeypot_name: "new_honeypot"
honeypot_config: "{{ honeypots.new_honeypot }}"
honeypot_compose_template: "docker-compose.yml.j2"
```

### Step 3: Create meta/main.yml

```yaml
---
dependencies:
  - role: honeypot_base
```

### Step 4: Create tasks/main.yml

```yaml
---
- name: Set honeypot variables
  set_fact:
    honeypot_name: "new_honeypot"
    honeypot_config: "{{ honeypots.new_honeypot }}"
    honeypot_compose_template: "docker-compose.yml.j2"

- name: Deploy honeypot
  include_role:
    name: honeypot_base
    tasks_from: deploy.yml
  when: honeypots.new_honeypot.enabled | default(true)
```

### Step 5: Create templates/docker-compose.yml.j2

```yaml
version: '3.8'

services:
  new_honeypot:
    image: {{ honeypot_config.image }}
    container_name: {{ honeypot_name }}
    restart: {{ honeypot_config.restart_policy | default('unless-stopped') }}
    
    ports:
{% for port in honeypot_config.ports %}
      - "{{ port.host }}:{{ port.container }}/{{ port.protocol }}"
{% endfor %}
    
    volumes:
      - {{ honeypot_data_dir }}:/data
      - {{ honeypot_logs_dir }}:/logs
      - {{ honeypot_config_dir }}:/config
    
    networks:
      - {{ honeypot_config.network_name }}
    
{% if honeypot_config.resource_limits is defined %}
    deploy:
      resources:
        limits:
          cpus: '{{ honeypot_config.resource_limits.cpus }}'
          memory: {{ honeypot_config.resource_limits.memory }}
{% endif %}

networks:
  {{ honeypot_config.network_name }}:
    external: true
```

### Step 6: Register in group_vars/all.yml

```yaml
honeypots:
  # ... existing honeypots ...
  
  new_honeypot:
    enabled: true
    description: "Description of the honeypot"
    image: "dockerhub/image:tag"
    ports:
      - { host: 9999, container: 9999, protocol: "tcp" }
    volumes: []
    environment: {}
    resource_limits:
      cpus: "0.5"
      memory: "512M"
    network_name: "honeypot_new_honeypot"
    healthcheck:
      test: "nc -z localhost 9999"
      interval: "30s"
      timeout: "10s"
      retries: 3
    restart_policy: "unless-stopped"
```

### Step 7: Deploy

```bash
ansible-playbook playbooks/deploy.yml --tags new_honeypot
```

That's it! Everything else (systemd service, logrotate, firewall rules, backup, uninstall) is automatically inherited from `honeypot_base`.

## ⚙️ Configuration Reference

### Global Settings (group_vars/all.yml)

#### Honeypot Registry

Each honeypot in the registry follows this structure:

```yaml
honeypots:
  honeypot_name:
    enabled: true                    # Enable/disable honeypot
    description: "Description"       # Human-readable description
    image: "docker/image:tag"        # Docker image
    ports:                           # Port mappings
      - { host: 2222, container: 2222, protocol: "tcp" }
      - { host: 161, container: 161, protocol: "udp" }
    volumes: []                      # Additional volumes (optional)
    environment:                     # Environment variables (optional)
      VAR_NAME: "value"
    resource_limits:                 # CPU/Memory limits
      cpus: "0.5"
      memory: "512M"
    network_name: "honeypot_name"   # Docker network name
    healthcheck:                     # Health check configuration
      test: "nc -z localhost 2222"
      interval: "30s"
      timeout: "10s"
      retries: 3
    restart_policy: "unless-stopped" # Docker restart policy
```

#### Backup Configuration

```yaml
backup:
  enabled: true
  directory: "/var/backups/honeypots"
  retention_days: 30
  compress: true
```

#### Logging Configuration

```yaml
logging:
  retention_days: 90
  rotate_size: "100M"
  rotate_count: 10
```

#### Firewall Configuration

```yaml
firewall:
  enabled: true
  default_policy: "deny"
  allow_ssh: true
  ssh_port: 22
```

## 🔍 Troubleshooting

### Docker Issues

```bash
# Check Docker service
sudo systemctl status docker

# View Docker logs
sudo journalctl -u docker -f

# Check container status
sudo docker ps -a
```

### Honeypot Not Starting

```bash
# Check systemd service
sudo systemctl status honeypot-cowrie

# View service logs
sudo journalctl -u honeypot-cowrie -f

# Check Docker Compose logs
cd /opt/cowrie
sudo docker compose logs -f
```

### Port Already in Use

```bash
# Check what's using the port
sudo netstat -tulpn | grep :2222
sudo lsof -i :2222

# Stop conflicting service
sudo systemctl stop <service-name>
```

### Firewall Blocking Connections

```bash
# Check firewall status (firewalld)
sudo firewall-cmd --list-all

# Check firewall status (ufw)
sudo ufw status verbose

# Check firewall status (iptables)
sudo iptables -L -n -v
```

### SELinux Issues (RHEL-based)

```bash
# Check SELinux status
getenforce

# View SELinux denials
sudo ausearch -m avc -ts recent

# Set SELinux to permissive temporarily
sudo setenforce 0
```

### Ansible Connection Issues

```bash
# Test connectivity
ansible honeypot_servers -m ping

# Test with verbose output
ansible-playbook playbooks/deploy.yml -vvv

# Test specific host
ansible honeypot-01 -m ping
```

## 🔒 Security Considerations

### 1. Honeypot Isolation

- Each honeypot runs in its own Docker network
- Configure host firewall to restrict access appropriately
- Consider running honeypots in isolated VLANs or DMZ

### 2. SSH Access

- Change default SSH port in production
- Use SSH key authentication, disable password auth
- Configure fail2ban for SSH protection
- Update `firewall.ssh_port` in `group_vars/all.yml`

### 3. Log Management

- Regularly review honeypot logs for suspicious activity
- Consider shipping logs to SIEM (ELK stack integration planned)
- Ensure sufficient disk space for logs

### 4. Updates

- Regularly update Docker images: `ansible-playbook playbooks/update.yml`
- Keep host OS patched and updated
- Monitor Docker and Ansible security advisories

### 5. Data Protection

- Backup honeypot data regularly: `ansible-playbook playbooks/backup.yml`
- Store backups on separate storage
- Test restore procedures periodically

### 6. Resource Limits

- Docker resource limits prevent DoS
- Monitor system resources (CPU, RAM, disk)
- Adjust `resource_limits` in config as needed

### 7. Port Exposure

- Only expose necessary ports to the internet
- Consider port knocking for SSH
- Use jump hosts for internal access

## 📚 Advanced Topics

### Multi-Host Deployment

Deploy to multiple hosts simultaneously:

```yaml
# inventory/hosts.yml
all:
  children:
    honeypot_servers:
      hosts:
        honeypot-01:
          ansible_host: 192.168.1.100
        honeypot-02:
          ansible_host: 192.168.1.101
        honeypot-03:
          ansible_host: 192.168.1.102
```

### Custom Port Mappings

Override default ports for specific deployments:

```yaml
# group_vars/all.yml or host_vars/honeypot-01.yml
honeypots:
  cowrie:
    ports:
      - { host: 22, container: 2222, protocol: "tcp" }  # Map to real SSH port
      - { host: 23, container: 2223, protocol: "tcp" }  # Map to real Telnet port
```

### Integration with ELK Stack

Configure centralized logging (coming soon):

```yaml
elk:
  enabled: true
  elasticsearch_host: "elk.example.com"
  elasticsearch_port: 9200
  logstash_host: "elk.example.com"
  logstash_port: 5044
```

### Custom Environment Variables

Add honeypot-specific environment variables:

```yaml
honeypots:
  cowrie:
    environment:
      COWRIE_HOSTNAME: "server-prod-01"
      COWRIE_LOG_LEVEL: "DEBUG"
      CUSTOM_VAR: "custom_value"
```

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Add New Honeypots**: Follow the pattern in "Adding a New Honeypot"
2. **Improve Documentation**: Fix typos, add examples
3. **Report Bugs**: Open an issue with details and reproduction steps
4. **Submit Pull Requests**: Fork, branch, commit, and open a PR

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- Honeypot projects: Cowrie, Dionaea, Honeytrap, Conpot, Elasticpot
- The Honeynet Project
- Docker and Ansible communities

## 📧 Support

- **Issues**: [GitHub Issues](https://github.com/oncumuhammed/mercan/issues)
- **Discussions**: [GitHub Discussions](https://github.com/oncumuhammed/mercan/discussions)

---

**Built with ❤️ for the security community**