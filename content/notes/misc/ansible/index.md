---
title: Ansible
menu:
  notes:
    name: Ansible
    identifier: notes-ansible
    parent: notes-miscellaneous
    weight: 20
---
# Ansible Installation
<!-- Ansible Installation -->
{{< note title="Ansible Installation: Control Node">}}
- [Docs](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#managed-node-requirements)
- Install `python3`: `apt install python3 python3-pip`
- Install `ansible`: `pip3 install ansible` 
  - or `python3 -m pip install --user ansible` 
  - or `python3 -m pip install --user ansible-core==2.12.3`
- Confirm: `ansible --version`
    - or `python3 -m pip show ansible`
{{< /note >}}
{{< note title="Ansible Installation: Managed node">}}
- Requires `Python 2.7`, or `Python 3.5 - 3.11` to run `Ansible` library code
- Install `python3`: `apt install python3 python3-pip`
- Needs a `user account` that can `SSH` to the node with an interactive `POSIX` shell
{{< /note >}}
{{< note title="Setting up Ansible: Inventory file">}}
{{< /note >}}

