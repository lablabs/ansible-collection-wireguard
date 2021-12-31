# Wireguard Server Ansible Collection

[![Molecule test](https://github.com/lablabs/ansible-collection-wireguard/actions/workflows/molecule.yaml/badge.svg?event=schedule)](https://github.com/lablabs/ansible-collection-wireguard/actions/workflows/molecule.yaml)

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
# Directory to store WireGuard configuration on the remote hosts
wireguard_dir: /etc/wireguard
wireguard_clients_dir: "{{ wireguard_dir }}/clients"

wireguard_clients_download_dir: clients/
wireguard_download_clients: false

# Predefined wireguard keys, this usually should be defined in ansible-vault
wireguard_privatekey_path: "{{ wireguard_dir }}/privatekey"
wireguard_publickey_path: "{{ wireguard_dir }}/publickey"
wireguard_presharedkey_path: "{{ wireguard_dir }}/presharedkey"

wireguard_systemd_path: /etc/systemd/network

# Wireguard packages
wireguard_repo_url: "{{ _repo_url }}"
wireguard_distro_packages: "{{ _distro_packages }}"

wireguard_packages:
  - wireguard-dkms
  - wireguard-tools

# The default port WireGuard will listen if not specified otherwise.
wireguard_port: 51820

# Client destination Hostname
wireguard_hostname: "{{ inventory_hostname }}"

# The default interface name that wireguard should use if not specified otherwise.
wireguard_interface: wg0

# Base wireguard subnet
wireguard_address: 10.213.213.0/24

wireguard_server_ip: "{{ wireguard_address | ipaddr('network') | ipmath(1) }}"
wireguard_subnetmask: "{{ wireguard_address | ipaddr('prefix') }}"

# XXX: This role only works with PrivateKeyFile/PresharedKeyFile it doesn't support variables.
wireguard_systemd_netdev:
  - NetDev:
      - Name: "{{ wireguard_interface }}"
      - Kind: wireguard
      - Description: "wireguard server: {{ wireguard_interface}} server on {{ wireguard_address }}"
  - WireGuard:
      - PrivateKey: "{{ _privkey_value['content'] | b64decode }}"
      - ListenPort: "{{ wireguard_port }}"

wireguard_systemd_network:
  - Match:
      - Name: "{{ wireguard_interface }}"
  - Network:
      - Address: "{{ wireguard_server_ip }}/{{ wireguard_subnetmask }}"
  - Route:
      - Destination: "{{ wireguard_address }}"
      - Gateway: "{{ wireguard_server_ip }}"

wireguard_keepalive: 25

wireguard_peers_allowed_ips: "{{ ([(_wireguard_interface_addr | ipaddr('net'))] + wireguard_additional_routes) | join(\", \") }}"
wireguard_peers: []
  # - name: user1
  #   allowed_ip: "10.213.213.2/32"
  #   publickey: "asdasdasdadsasdasd"
  # - name: user2
  #   allowed_ip: "10.213.213.3/32"
  #   publickey: "000000000000000000"
  #   keepalive: 30
  # - name: user3
  #   allowed_ip: "10.213.213.4/32"
  #   publickey: "111111111111111111"
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