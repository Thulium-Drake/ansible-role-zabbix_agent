[![Build Status](https://drone.element-networks.nl/api/badges/Element-Networks/ansible-role_zabbix-agent/status.svg)](https://drone.element-networks.nl/Element-Networks/ansible-role_zabbix-agent)

# Zabbix agent as configured by Ansible
This role can configure an Zabbix agent on your Ansible managed hosts.

It will install an agent and configure it for use with a Zabbix server using
PSK encrypted connections. The keys required for the connection are generated
using a deterministic approach, so they don't have to be saved on the ansible
controller.

This role is also able to deploy supporting scripts for Zabbix templates such as
the ones in https://github.com/Thulium-Drake/zabbix-templates.

If you have your own collection of Zabbix templates, you can still use this role if
you place your template scripts and configs in a similar structure.

## Setup
Plugin requirements:
  * Ansible Merge Vars: https://github.com/leapfrogonline/ansible-merge-vars
Collection requirements:
  * community.crypto
  * community.zabbix
Windows collection requirements:
  * ansible.windows
  * community.windows
Incorporate the role in your regular Ansible playbooks and configure it.

## Windows Setup
When installing Zabbix on Windows targets, please note that this role assumes the msi package is
present on the Ansible Controller. Please set the variable for which version you wish to install
(default is 5.0.4). Then place the MSI package at:

```
<playbooks directory>/zabbix_agent-5.0.4-windows-amd64-openssl.msi
```

Then run this role against your clients with the following play (add variables
required to connect to the Windows host via WinRM or SSH):

```
---
- name: 'Setup Zabbix agent on Windows'
  hosts: 'win_clients'
  tasks:
    - name: 'Run role'
      import_role:
        name: 'zabbix_agent'
        tasks_from: 'win_main.yml'
```

## Zabbix Server setup
This role can also be used to install the Zabbix server on a system. Please note that
that this role does _not_ install or configure a database server for you. The author
recommends to use geerlingguy.mysql to set up the MariaDB server.

For detailed instructions, please check the defaults file.

### SELinux on the Server
This has not been fully worked out yet, but installing a Zabbix server with SELinux enabled is a
minor challenge. So far I've made it work using the following steps:

1. ```setsebool -P zabbix_can_network 1```
2. ```setsebool -P httpd_can_connect_zabbix 1```
3. ```cd; grep "denied.*zabbix" /var/log/audit/audit.log | audit2allow -M zabbix_policy; semodule -i zabbix_policy.pp```
