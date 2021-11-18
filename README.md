# Wireguard Server Ansible Collection

[![Ansible-lint](https://github.com/lablabs/ansible-collection-wireguard/actions/workflows/lint.yaml/badge.svg)](https://github.com/lablabs/ansible-collection-wireguard/actions)

[<img src="ll-logo.png">](https://lablabs.io/)

Wireguard VPN Server Ansible Collection.

This collection will:

1. Prepare data disk and mount it to the defined mountpoint (playbook name `disk`)
2. Run SSH and Linux OS haredning tasks  (playbook name `security`)
3. Install and configure fail2ban (playbook name `security`)
4. Install and configure wireguard (playbook name `wireguard`)

## How to run Ansible playbooks from this collection

First make sure your future destination host is up and running and you have an access to SSH Private Key file.  
Prepare the Python prerequisites for Ansible Roles in this Collecion:

```bash
pip install -r requirements.txt
```

Run Ansible to configure the [Wireguard](https://www.wireguard.com/) server:

```bash
ansible-playbook lablabs.wireguard.disk -i <inventory> --private-key <private key file>
ansible-playbook lablabs.wireguard.security -i <inventory> --private-key <private key file>
ansible-playbook lablabs.wireguard.wireguard -i <inventory> --private-key <private key file>
```

By default all 3 playbooks target `all` hosts in the inventory. You can target specific host or group by using variable `target`:

```bash
ansible-playbook lablabs.wireguard.wireguard -e taget=wireguard -i <inventory> --private-key <private key file>
```

You may want to create a playbook to run all 3 playbooks in one run:

```yaml
- name: Prepare Data disk
  import_playbook: lablabs.wireguard.disk
  tags: disk

- name: Run Security hardening
  import_playbook: lablabs.wireguard.security
  tags: security

- name: Install and configure Wireguard
  import_playbook: lablabs.wireguard.wireguard
  tags: wireguard
```

## Variables example

```yaml
---
###
# Data disk config
configure_data_disk: true
data_disk: /dev/nvme1n1
data_disk_partition: /dev/nvme1n1p1
data_disk_mount: /data

###
# Security hardening

# SSH
sftp_enabled: true
ssh_max_auth_retries: 6
ssh_permit_tunnel: true
ssh_allow_tcp_forwarding: true
ssh_allow_agent_forwarding: true
ssh_client_alive_count: 100
ssh_client_alive_interval: 40

# OS
sysctl_overwrite:
  net.ipv4.ip_forward: 1

###
# Wireguard deployment
wireguard_dir: /data/wireguard
wireguard_address: 10.214.214.0/24

wireguard_clients_download_dir: "{{ lookup('env', 'PWD') }}/clients/"
wireguard_download_clients: true

wireguard_hostname: "{{ inventory_hostname }}"

wireguard_out_interface: ens5

wireguard_additional_routes:
  - 172.11.0.0/16
  - 172.22.0.0/20
  - 172.33.0.0/16

# concat routes defined on ens5 brigde and additional ones
_wireguard_interface_addr: "{{ ansible_ens5.ipv4.network }}/{{ ansible_ens5.ipv4.netmask }}"
wireguard_peers_allowed_ips: "{{ ([(_wireguard_interface_addr | ipaddr('net'))] + wireguard_additional_routes) | join(\", \") }}"

wireguard_peers:
  - name: client1
    allowed_ip: 10.214.214.101
    publickey: "12345678900987654321"
  - name: client2
    allowed_ip: 10.214.214.102
    publickey: "qwertyuuiopoiuytrewq"
  - name: client3
    allowed_ip: 10.214.214.103
    publickey: "zxcvbnmzxcvbnmzxc"
```

## License

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

See [LICENSE](LICENSE) for full details.

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

## Author Information

Created in 2021 by [Labyrinth Labs](https://www.lablabs.io/)