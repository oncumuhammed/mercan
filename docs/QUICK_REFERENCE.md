# Mercan Quick Reference Guide

## Quick Commands

### Deploy Honeypots
```bash
# Deploy all enabled honeypots
ansible-playbook playbooks/deploy.yml

# Deploy specific honeypot
ansible-playbook playbooks/deploy.yml --tags cowrie

# Deploy multiple honeypots
ansible-playbook playbooks/deploy.yml --tags cowrie,dionaea

# Skip infrastructure setup
ansible-playbook playbooks/deploy.yml --skip-tags infrastructure
```

### Update Honeypots
```bash
# Update all
ansible-playbook playbooks/update.yml

# Update specific
ansible-playbook playbooks/update.yml --tags dionaea
```

### Backup
```bash
# Backup all
ansible-playbook playbooks/backup.yml

# Backup specific
ansible-playbook playbooks/backup.yml --tags cowrie

# Restore from backup
tar -xzf /var/backups/honeypots/cowrie_20260211T120000.tar.gz -C /
```

### Status Check
```bash
# Check all
ansible-playbook playbooks/status.yml

# Check specific
ansible-playbook playbooks/status.yml --tags elasticpot
```

### Uninstall
```bash
# Uninstall but keep data
ansible-playbook playbooks/uninstall.yml

# Uninstall and remove data
ansible-playbook playbooks/uninstall.yml --extra-vars "remove_data=true"

# Skip confirmation
ansible-playbook playbooks/uninstall.yml --extra-vars "auto_approve=true"
```

## Systemd Service Commands

```bash
# Start honeypot
sudo systemctl start honeypot-cowrie

# Stop honeypot
sudo systemctl stop honeypot-cowrie

# Restart honeypot
sudo systemctl restart honeypot-cowrie

# Check status
sudo systemctl status honeypot-cowrie

# View logs
sudo journalctl -u honeypot-cowrie -f

# Enable auto-start
sudo systemctl enable honeypot-cowrie

# Disable auto-start
sudo systemctl disable honeypot-cowrie
```

## Docker Commands

```bash
# View running containers
cd /opt/cowrie
sudo docker compose ps

# View logs
sudo docker compose logs -f

# Restart container
sudo docker compose restart

# Stop container
sudo docker compose down

# Start container
sudo docker compose up -d

# Pull latest image
sudo docker compose pull
```

## File Locations

### Configuration
- **Main Config**: `group_vars/all.yml`
- **Inventory**: `inventory/hosts.yml`
- **Ansible Config**: `ansible.cfg`

### Honeypot Files
- **Data**: `/opt/<honeypot>/data/`
- **Logs**: `/opt/<honeypot>/logs/`
- **Config**: `/opt/<honeypot>/config/`
- **Docker Compose**: `/opt/<honeypot>/docker-compose.yml`

### System Files
- **Systemd**: `/etc/systemd/system/honeypot-<name>.service`
- **Logrotate**: `/etc/logrotate.d/<honeypot>`
- **Backups**: `/var/backups/honeypots/`

## Port Mappings

| Honeypot | Ports | Description |
|----------|-------|-------------|
| Cowrie | 2222, 2223 | SSH, Telnet |
| Dionaea | 21, 80, 443, 445, 1433, 3306, 5060, 11211 | Multi-protocol |
| Honeytrap | 8000-8010 | Advanced trap |
| Conpot | 8800, 102, 502, 161/UDP, 47808 | ICS/SCADA |
| Elasticpot | 9200 | Elasticsearch |

## Troubleshooting

### Check Docker Service
```bash
sudo systemctl status docker
sudo journalctl -u docker -f
```

### Check Firewall
```bash
# firewalld
sudo firewall-cmd --list-all

# ufw
sudo ufw status verbose

# iptables
sudo iptables -L -n -v
```

### Check SELinux (RHEL-based)
```bash
getenforce
sudo ausearch -m avc -ts recent
```

### Port Conflicts
```bash
sudo netstat -tulpn | grep :2222
sudo lsof -i :2222
```

### Ansible Connectivity
```bash
ansible honeypot_servers -m ping
ansible-playbook playbooks/deploy.yml -vvv
```

## Enable/Disable Honeypots

Edit `group_vars/all.yml`:
```yaml
honeypots:
  cowrie:
    enabled: true   # Change to false to disable
```

Then redeploy:
```bash
ansible-playbook playbooks/deploy.yml
```

## Adding New Honeypots

1. Create role directory: `mkdir -p roles/new_honeypot/{tasks,templates,defaults,meta}`
2. Copy template files from existing honeypot (e.g., cowrie)
3. Customize Docker Compose template
4. Add to `group_vars/all.yml`
5. Deploy: `ansible-playbook playbooks/deploy.yml --tags new_honeypot`

See `docs/ROLE_TEMPLATE.md` for detailed instructions.

## Common Tasks

### View All Logs
```bash
tail -f /opt/*/logs/*.log
```

### Check Disk Usage
```bash
du -sh /opt/*
df -h /opt
```

### List All Containers
```bash
docker ps -a
```

### Remove All Stopped Containers
```bash
docker container prune
```

### Update All Docker Images
```bash
ansible-playbook playbooks/update.yml
```

### Create Manual Backup
```bash
ansible-playbook playbooks/backup.yml
```

### Check All Service Status
```bash
systemctl list-units "honeypot-*"
```
