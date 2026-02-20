# Security Remediation Playbooks

Playbooks для устранения уязвимостей и применения security-патчей.

## Структура

| Playbook | Назначение |
|----------|-----------|
| `patch-vulnerabilities.yml` | Полное обновление: системные пакеты + Docker образы |
| `patch-host-only.yml` | Только системные пакеты (kernel, openssl, zlib) |
| `patch-containers-only.yml` | Только Docker образы без рестарта хоста |

## Быстрый старт

### Полное обновление (рекомендуется)

```bash
ansible-playbook playbooks/remediation/patch-vulnerabilities.yml --ask-vault-pass
```

### Только критические системные пакеты

```bash
ansible-playbook playbooks/remediation/patch-host-only.yml --ask-vault-pass
```

### Только Docker образы (быстрее, без downtime хоста)

```bash
ansible-playbook playbooks/remediation/patch-containers-only.yml --ask-vault-pass
```

## Текущие известные уязвимости

### CRITICAL (требуют немедленного исправления)

| CVE | Компонент | Описание | Исправление |
|-----|-----------|----------|-------------|
| CVE-2025-68121 | Tracee, Docker | crypto/tls session resumption bypass | Обновить Go до 1.24.13+ |
| CVE-2017-18342 | Host Python | PyYAML RCE via yaml.load() | Обновить PyYAML до 5.4+ |
| CVE-2024-24790 | Go apps | IPv4-mapped IPv6 bypass | Обновить Go до 1.21.11+ |
| CVE-2024-25621 | Docker Buildx | containerd privilege escalation | Обновить containerd до 2.0.7+ |

### HIGH (рекомендуется исправить в течение недели)

| CVE | Компонент | Описание |
|-----|-----------|----------|
| CVE-2025-61728 | Go stdlib | DoS via archive/zip |
| CVE-2025-61729 | Go stdlib | DoS via crypto/x509 |
| CVE-2025-22869 | golang.org/x/crypto | SSH key exchange DoS |

## Мониторинг после исправления

1. **Grafana Dashboard**: `Security Findings` → проверить снижение счетчиков
2. **Trivy метрики**: `/var/lib/node_exporter/textfile_collector/trivy.prom`
3. **Логи**: `/var/lib/trivy/logs/trivy-scan.log`

```bash
# Проверить количество критических уязвимостей
ssh vpn@<domain> "grep -c 'CRITICAL' /var/lib/trivy/logs/trivy-scan.log"

# Топ CVE после обновления
ssh vpn@<domain> "grep CRITICAL /var/lib/trivy/logs/trivy-scan.log | jq -r '.cve' | sort | uniq -c | sort -rn | head -10"
```

## Безопасность выполнения

- Все playbooks запрашивают подтверждение перед началом
- Создаются бэкапы конфигураций
- Проверяется healthcheck сервисов после рестарта
- Поддерживается `--check` mode для dry-run
