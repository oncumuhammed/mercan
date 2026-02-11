# Example: Deploying Mercan Honeypots

This guide walks through a complete deployment example from start to finish.

## Scenario

Deploy Cowrie and Dionaea honeypots on a fresh Ubuntu 22.04 server.

## Prerequisites

- Ubuntu 22.04 server (192.168.1.100)
- SSH access with sudo privileges
- Ansible installed on your control machine

## Step 1: Clone Repository

```bash
git clone https://github.com/oncumuhammed/mercan.git
cd mercan
```

## Step 2: Configure Inventory

Edit `inventory/hosts.yml`:

```yaml
---
all:
  children:
    honeypot_servers:
      hosts:
        honeypot-prod:
          ansible_host: 192.168.1.100
          ansible_user: ubuntu
          ansible_become: yes
  
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

## Step 3: Verify Connectivity

```bash
ansible honeypot_servers -m ping
```

Expected output:
```
honeypot-prod | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Step 4: Review Configuration

Check `group_vars/all.yml`:

```yaml
honeypots:
  cowrie:
    enabled: true  # ✓ Enabled
    # ... configuration ...
  
  dionaea:
    enabled: true  # ✓ Enabled
    # ... configuration ...
  
  honeytrap:
    enabled: false  # ✗ Disabled for this deployment
  
  conpot:
    enabled: false  # ✗ Disabled
  
  elasticpot:
    enabled: false  # ✗ Disabled
```

## Step 5: Dry Run Deployment

Test without making changes:

```bash
ansible-playbook playbooks/deploy.yml --check
```

## Step 6: Deploy Infrastructure

Deploy common packages and Docker:

```bash
ansible-playbook playbooks/deploy.yml --tags infrastructure
```

This will:
- Install required packages (curl, git, etc.)
- Configure firewall
- Handle SELinux (if applicable)
- Install Docker and Docker Compose
- Start Docker service

Expected duration: 2-5 minutes

## Step 7: Deploy Honeypots

Deploy Cowrie and Dionaea:

```bash
ansible-playbook playbooks/deploy.yml --tags honeypots
```

Or deploy individually:

```bash
# Deploy only Cowrie
ansible-playbook playbooks/deploy.yml --tags cowrie

# Deploy only Dionaea
ansible-playbook playbooks/deploy.yml --tags dionaea
```

This will:
- Create directory structure
- Create Docker networks
- Deploy Docker Compose files
- Create systemd services
- Configure logrotate
- Open firewall ports
- Start containers
- Enable auto-start on boot

Expected duration: 3-5 minutes per honeypot

## Step 8: Verify Deployment

Check status:

```bash
ansible-playbook playbooks/status.yml
```

Check on the server:

```bash
# Check systemd services
ssh ubuntu@192.168.1.100 "systemctl status honeypot-cowrie"
ssh ubuntu@192.168.1.100 "systemctl status honeypot-dionaea"

# Check Docker containers
ssh ubuntu@192.168.1.100 "docker ps"

# Check directories
ssh ubuntu@192.168.1.100 "ls -la /opt/"

# View logs
ssh ubuntu@192.168.1.100 "tail -f /opt/cowrie/logs/*.log"
```

## Step 9: Test Honeypots

Test Cowrie (SSH):
```bash
ssh -p 2222 ubuntu@192.168.1.100
# Try common credentials: root/password, admin/admin
```

Test Dionaea (HTTP):
```bash
curl http://192.168.1.100:80
curl http://192.168.1.100:443 -k
```

## Step 10: Monitor

View real-time logs:

```bash
# Cowrie logs
ssh ubuntu@192.168.1.100 "tail -f /opt/cowrie/logs/*.log"

# Dionaea logs
ssh ubuntu@192.168.1.100 "tail -f /opt/dionaea/logs/*.log"

# Systemd logs
ssh ubuntu@192.168.1.100 "journalctl -u honeypot-cowrie -f"
```

## Step 11: Create Backup

```bash
ansible-playbook playbooks/backup.yml
```

Backups stored in: `/var/backups/honeypots/`

## Step 12: Update Honeypots

When new Docker images are available:

```bash
ansible-playbook playbooks/update.yml
```

Or update specific honeypot:

```bash
ansible-playbook playbooks/update.yml --tags cowrie
```

## Troubleshooting Common Issues

### Issue: Container fails to start

**Check logs:**
```bash
ssh ubuntu@192.168.1.100 "cd /opt/cowrie && docker compose logs"
```

**Common causes:**
- Port already in use
- Insufficient permissions
- SELinux blocking

### Issue: Firewall blocking connections

**Check firewall:**
```bash
# Ubuntu (ufw)
ssh ubuntu@192.168.1.100 "sudo ufw status verbose"

# CentOS/RHEL (firewalld)
ssh ubuntu@192.168.1.100 "sudo firewall-cmd --list-all"
```

**Manual fix:**
```bash
# Ubuntu
ssh ubuntu@192.168.1.100 "sudo ufw allow 2222/tcp"

# CentOS/RHEL
ssh ubuntu@192.168.1.100 "sudo firewall-cmd --permanent --add-port=2222/tcp"
ssh ubuntu@192.168.1.100 "sudo firewall-cmd --reload"
```

### Issue: Service won't start on boot

**Check service status:**
```bash
ssh ubuntu@192.168.1.100 "systemctl is-enabled honeypot-cowrie"
```

**Enable manually:**
```bash
ssh ubuntu@192.168.1.100 "sudo systemctl enable honeypot-cowrie"
```

## Advanced: Multi-Server Deployment

Deploy to multiple servers:

Edit `inventory/hosts.yml`:

```yaml
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

Deploy to all:
```bash
ansible-playbook playbooks/deploy.yml
```

Deploy to specific host:
```bash
ansible-playbook playbooks/deploy.yml --limit honeypot-01
```

## Cleaning Up

Uninstall honeypots (keep data):
```bash
ansible-playbook playbooks/uninstall.yml
```

Uninstall and remove data:
```bash
ansible-playbook playbooks/uninstall.yml --extra-vars "remove_data=true"
```

## Summary

You've successfully:
- ✅ Deployed infrastructure (Docker, packages)
- ✅ Deployed two honeypots (Cowrie, Dionaea)
- ✅ Configured systemd services
- ✅ Set up firewall rules
- ✅ Created backups
- ✅ Monitored honeypot activity

## Next Steps

1. **Monitor regularly**: Check logs for attacks
2. **Update frequently**: Keep Docker images current
3. **Backup regularly**: Schedule weekly backups
4. **Add more honeypots**: Follow the role template
5. **Integrate with SIEM**: Ship logs to ELK stack (coming soon)

## Resources

- **Main Documentation**: README.md
- **Role Template**: docs/ROLE_TEMPLATE.md
- **Quick Reference**: docs/QUICK_REFERENCE.md
- **Contributing**: CONTRIBUTING.md
