# Ansible Collection - homelab.mylab

Overview
--------
The homelab.mylab Ansible Collection provides playbooks and roles to automate provisioning of a homelab environment.
It includes tasks to:

* Provision and configure virtual machines

* Deploy Ansible Automation Platform 2.5 (AAP)

This collection is designed for lab, testing, and learning scenarios where you want a repeatable environment setup

Installation
------------

* From local tarball
# ansible-galaxy collection install homelab-mylab-1.0.0.tar.gz

* From Git (development)
requirements.yml
# ansible-galaxy collection install -r requirements.yml

* After installation, playbooks and roles will be available under:
# ~/.ansible/collections/ansible_collections/homelab/mylab/

* Collection Layout
playbooks/
  ├── site.yml         # Main entrypoint
  ├── deployvms.yml    # VM provisioning
  ├── aap25.yml        # AAP deployment
  └── inventories/
      └── hosts.yml    # Dynamic inventory

roles/
  ├── deploy_vms/      # Role to provision virtual machines
  ├── aap25_deploy/    # Role to install Ansible Automation Platform 2.5

* USAGE

* Run only VM provisioning
ansible-playbook homelab.mylab.playbooks.site.yml --tags deployvms -e target_env=aap

* Run only AAP deployment
ansible-playbook homelab.mylab.playbooks.site.yml --tags aap25 -e target_env=aap

* Requirements

# Ansible >= 2.13
# Python >= 3.9
# SSH access to target hosts or virtualization environment

Dependencies defined in galaxy.yml (if any)

* Author
Romulo Macedo (@rcmacedo)
