# Wireguard Server Ansible Collection

[![Molecule test](https://github.com/lablabs/ansible-collection-wireguard/actions/workflows/molecule.yaml/badge.svg)](https://github.com/lablabs/ansible-collection-wireguard/actions/workflows/molecule.yaml)

[<img src="ll-logo.png">](https://lablabs.io/)

Wireguard VPN Server Ansible Collection.

This collection will:

1. Prepare data disk and mount it to the defined mountpoint (playbook name `disk`)
2. Run SSH and Linux OS haredning tasks  (playbook name `security`)
3. Install and configure fail2ban (playbook name `security`)
4. Install and configure wireguard (playbook name `wireguard`)

## Supported (tested) Linux Distributions

- Rocky Linux 8
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS

### Eventually supported but not tested

- Debian 11 is supported but the system must use `systemd-networkd` networking service instead of `Network Manager`.

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

## Variables

[Wireguard Role Variables](roles/wireguard/defaults/main.yml)

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
