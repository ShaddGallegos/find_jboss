# Find JBoss Network Scanner

This project scans a network segment and reports how many discovered systems appear to have Red Hat JBoss or EAP artifacts.

## What this does

- Discovers live hosts in a target CIDR using nmap.
- Attempts SSH checks against each discovered host.
- Detects JBoss/EAP artifacts by checking:
  - installed packages
  - running processes
  - systemd unit names
  - common install directories
- Prints a summary including:
  - total discovered hosts
  - hosts with JBoss artifacts
  - hosts without JBoss artifacts
  - hosts unreachable via SSH
  - list of hosts where JBoss artifacts were found

## Folder structure

- ansible.cfg
- playbooks/scan_jboss_network.yml
- roles/jboss_network_scan/defaults/main.yml
- roles/jboss_network_scan/tasks/main.yml

## Prerequisites

- Ansible installed on the scanner host.
- Scanner host has nmap and nc installed.
- SSH access from scanner host to target systems (key-based access recommended).

Example package install on RHEL-like systems:

    sudo dnf install -y nmap nmap-ncat

## How to run

1. Change into the project directory.

    cd /home/sgallego/GIT/find_jboss

2. Run the scan with defaults.

    ansible-playbook -i localhost, playbooks/scan_jboss_network.yml

Default network is 192.168.0.0/24.

## Common runtime overrides

Scan a specific subnet:

    ansible-playbook -i localhost, playbooks/scan_jboss_network.yml \
      -e jboss_scan_network_cidr=192.168.0.0/24

Set SSH user and private key:

    ansible-playbook -i localhost, playbooks/scan_jboss_network.yml \
      -e jboss_scan_ssh_user=admin \
      -e jboss_scan_ssh_private_key=~/.ssh/id_rsa

Change SSH timeout and port:

    ansible-playbook -i localhost, playbooks/scan_jboss_network.yml \
      -e jboss_scan_ssh_port=22 \
      -e jboss_scan_connect_timeout=8



## Key variables

Defaults are defined in roles/jboss_network_scan/defaults/main.yml.

- jboss_scan_network_cidr: 192.168.0.0/24
- jboss_scan_ssh_user: current shell user
- jboss_scan_ssh_port: 22
- jboss_scan_ssh_private_key: empty
- jboss_scan_connect_timeout: 5
- jboss_scan_require_ssh_port_open: true
- jboss_scan_extra_ssh_options: empty

## How to read results

At the end of the run, look for the summary block labeled:

- JBoss scan summary for <network>

The most important line for your goal is:

- Hosts with JBoss artifacts: <count>

That value is the number of discovered systems where JBoss signals were detected.

## Troubleshooting

No hosts discovered:

- Confirm routing and network reachability from scanner host.
- Verify CIDR is correct.
- Check local firewall rules.

Hosts discovered but all unreachable via SSH:

- Validate SSH key and username.
- Ensure target SSH service is running and reachable on the configured port.
- Set jboss_scan_require_ssh_port_open=false if you want SSH attempts even when nc port probe fails.

Permission denied during SSH checks:

- Use a user with enough privileges to inspect services/processes/packages.
- Consider adding sudo logic in the role if your environment requires it.
