# Honeypot Role Template Documentation

This document provides a complete guide for creating new honeypot roles following the established modular pattern in the Mercan framework.

## Overview

The Mercan framework uses a **highly modular architecture** where:
1. Each honeypot is an independent Ansible role
2. All honeypot roles follow an **identical structure**
3. Common functionality is inherited from the `honeypot_base` role
4. Honeypots are registered in a central configuration file
5. Playbooks dynamically iterate over registered honeypots

## Role Structure Template

Every honeypot role follows this exact directory structure:

```
roles/honeypot_name/
├── defaults/
│   └── main.yml           # Default variables
├── meta/
│   └── main.yml           # Role metadata and dependencies
├── tasks/
│   └── main.yml           # Main task file
└── templates/
    └── docker-compose.yml.j2  # Docker Compose template
```

## Step-by-Step Guide

### Step 1: Create Role Directory Structure

```bash
cd /path/to/mercan
mkdir -p roles/your_honeypot/{tasks,templates,defaults,meta}
```

Replace `your_honeypot` with your honeypot name (lowercase, underscores for spaces).

### Step 2: Create defaults/main.yml

This file sets the honeypot name and references the configuration from the registry.

**Template:**
```yaml
---
honeypot_name: "your_honeypot"
honeypot_config: "{{ honeypots.your_honeypot }}"
honeypot_compose_template: "docker-compose.yml.j2"
```

### Step 3: Create meta/main.yml

This file declares the dependency on `honeypot_base`.

**Template:**
```yaml
---
dependencies:
  - role: honeypot_base
```

### Step 4: Create tasks/main.yml

**Template:**
```yaml
---
- name: Set honeypot variables
  set_fact:
    honeypot_name: "your_honeypot"
    honeypot_config: "{{ honeypots.your_honeypot }}"
    honeypot_compose_template: "docker-compose.yml.j2"

- name: Deploy honeypot
  include_role:
    name: honeypot_base
    tasks_from: deploy.yml
  when: honeypots.your_honeypot.enabled | default(true)
```

### Step 5: Create templates/docker-compose.yml.j2

**Template:**
```yaml
version: '3.8'

services:
  your_honeypot:
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
  your_honeypot:
    enabled: true
    description: "Brief description"
    image: "dockerhub/image:latest"
    ports:
      - { host: 9999, container: 9999, protocol: "tcp" }
    volumes: []
    environment: {}
    resource_limits:
      cpus: "0.5"
      memory: "512M"
    network_name: "honeypot_your_honeypot"
    healthcheck:
      test: "nc -z localhost 9999"
      interval: "30s"
      timeout: "10s"
      retries: 3
    restart_policy: "unless-stopped"
```

### Step 7: Deploy

```bash
ansible-playbook playbooks/deploy.yml --tags your_honeypot
```

## What You Get Automatically

- **Directory Structure**: `/opt/your_honeypot/{data,logs,config}`
- **Systemd Service**: `honeypot-your_honeypot` with auto-start
- **Docker Network**: Isolated network per honeypot
- **Firewall Rules**: Automatic configuration
- **Log Rotation**: Daily rotation with retention
- **Backup Support**: Automatic inclusion in backups
- **Management**: All playbooks work automatically

## Best Practices

1. **Naming**: Use lowercase with underscores
2. **Ports**: Check for conflicts
3. **Resources**: Start with 0.5 CPU, 512M RAM
4. **Health Checks**: Keep simple and fast
5. **Volumes**: Match Docker image paths
6. **Documentation**: Comment your templates

For complete examples and detailed documentation, see the README.md file.
