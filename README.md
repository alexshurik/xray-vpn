# xray-vpn

Ansible automation for deploying a VPN server based on [3x-ui](https://github.com/MHSanaei/3x-ui) with VLESS + Reality protocol.

## What gets deployed

- **3x-ui** panel (Docker, host network) with VLESS Reality
- **Let's Encrypt** SSL certificate via certbot
- **UFW** firewall with minimal open ports
- **Hardening** — fail2ban, unattended-upgrades, watchtower, SSH keys
- **Monitoring** (optional) — Grafana, Prometheus, Loki, Promtail, node-exporter
- **Security monitoring** (optional) — Falco, Tracee, Trivy

## Prerequisites

- Python 3.9+
- Ansible:
  ```bash
  pip install ansible-core>=2.15
  ```
- VPS with Ubuntu 22.04/24.04 and root access (password or SSH key)

## Quick start

### 1. Clone and install dependencies

```bash
git clone https://github.com/alexshurik/xray-vpn.git
cd xray-vpn
ansible-galaxy collection install -r requirements.yml
```

### 2. Bootstrap (new server)

Connects to VPS, deploys SSH key, creates config files:

```bash
ansible-playbook --ask-pass playbooks/bootstrap.yml
```

You will be prompted for VPS IP, SSH credentials, domain, and email.

After bootstrap, edit passwords in `group_vars/all/vault.yml` and encrypt:

```bash
ansible-vault encrypt group_vars/all/vault.yml
```

**Alternative** — configure manually:

```bash
cp inventory/hosts.yml.example inventory/hosts.yml
cp group_vars/all/vars.yml.example group_vars/all/vars.yml
cp group_vars/all/vault.yml.example group_vars/all/vault.yml
# Edit all three files, then encrypt vault
ansible-vault encrypt group_vars/all/vault.yml
```

### 3. Deploy

```bash
ansible-playbook --ask-vault-pass playbooks/deploy.yml
```

With monitoring:

```bash
ansible-playbook --ask-vault-pass playbooks/deploy.yml \
  -e monitoring_enabled=true
```

With monitoring + security monitoring:

```bash
ansible-playbook --ask-vault-pass playbooks/deploy.yml \
  -e monitoring_enabled=true \
  -e security_monitoring_enabled=true
```

## Playbooks

| Playbook | Purpose |
|----------|---------|
| `bootstrap.yml` | First-time VPS setup: SSH key + config creation |
| `deploy.yml` | Main deployment (VPN, optional monitoring/security) |
| `deploy-monitoring.yml` | Deploy monitoring stack only (incremental) |
| `deploy-security.yml` | Deploy security monitoring only (requires monitoring) |
| `remediation/*.yml` | Vulnerability patching (see `playbooks/remediation/README.md`) |

## Access after deployment

| Service | URL | Login | Password |
|---------|-----|-------|----------|
| 3x-ui | `https://<domain>:2053` | `admin` | `vault_vpn_admin_password` |
| Grafana | `https://<domain>:3000` | `admin` | `vault_grafana_admin_password` |

## Ports

| Port | Service | External |
|------|---------|----------|
| 22 | SSH | yes |
| 80 | Let's Encrypt challenge | yes |
| 443 | HTTPS | yes |
| 2053 | 3x-ui panel | yes |
| 2096 | 3x-ui subscriptions | yes |
| 25000–25100 | VPN inbound range | yes |
| 3000 | Grafana | yes (if monitoring) |
| 9090 | Prometheus | localhost only |
| 3100 | Loki | localhost only |

## Secrets and git

Sensitive files are excluded from git via `.gitignore`:

- `inventory/hosts.yml` — real inventory
- `group_vars/all/vars.yml` — domain, email
- `group_vars/all/vault.yml` — encrypted passwords
- `.vault_pass` — vault password file

Only `.example` templates with placeholder values are tracked.
