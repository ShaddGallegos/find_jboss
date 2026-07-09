# jboss_network_scan role

Scans a network segment for reachable systems and checks each reachable host for Red Hat JBoss/EAP artifacts.

## What it reports

- Total discovered hosts
- Hosts with JBoss artifacts
- Hosts without JBoss artifacts
- Hosts unreachable via SSH
- List of hosts where JBoss artifacts were detected

## Detection signals

A host is considered to have JBoss artifacts if at least one signal is present:

- Installed package names matching `jboss` or `eap`
- Running processes matching `jboss` or `org.jboss`
- systemd units matching `jboss` or `eap`
- Known filesystem paths:
  - `/opt/jboss`
  - `/opt/rh/eap7`
  - `/opt/rh/eap8`
  - `/usr/share/jbossas`

## Defaults

Defined in `defaults/main.yml`:

- `jboss_scan_network_cidr: 192.168.0.0/24`
- `jboss_scan_ssh_user: {{ lookup('env', 'USER') }}`
- `jboss_scan_ssh_port: 22`
- `jboss_scan_ssh_private_key: ""`
- `jboss_scan_connect_timeout: 5`
- `jboss_scan_require_ssh_port_open: true`

## Run

From `aap_workflow_project`:

```bash
ansible-playbook -i localhost, playbooks/scan_jboss_network.yml
```

Override defaults at runtime:

```bash
ansible-playbook -i localhost, playbooks/scan_jboss_network.yml \
  -e jboss_scan_network_cidr=192.168.0.0/24 \
  -e jboss_scan_ssh_user=admin \
  -e jboss_scan_ssh_private_key=~/.ssh/id_rsa
```

## Notes

- The scanner host must have `nmap` and `nc` installed.
- SSH key-based access is recommended for non-interactive scanning.
