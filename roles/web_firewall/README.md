# Web Firewall Role

## Purpose

The `web_firewall` role adds firewall rules required by web servers.

The common role manages the baseline firewall configuration for every VM:

- deny incoming traffic by default;
- allow outgoing traffic by default;
- allow OpenSSH;
- enable UFW.

The `web_firewall` role adds only web-specific access.

---

## Managed ports

### HTTP

```text
TCP/80
```

Enabled by default.

### HTTPS

```text
TCP/443
```

Disabled until TLS is introduced.

---

## Variables

```yaml
web_firewall_http_port: 80
web_firewall_https_enabled: false
web_firewall_https_port: 443
```

---

## Security model

The web server should expose:

```text
22/tcp → OpenSSH
80/tcp → Nginx
```

Gunicorn remains bound to:

```text
127.0.0.1:8000
```

Port `8000` must not be allowed through UFW.

---

## Validation

```bash
ansible web01 \
  -i inventories/production/hosts.ini \
  -b \
  -m ansible.builtin.command \
  -a "ufw status verbose"
```
