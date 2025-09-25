# Ansible Collection: `homelab.mylab`

##  Overview
The **`homelab.mylab`** Ansible Collection provides playbooks and roles to automate provisioning of a homelab environment.  
It includes automation for:  

- Provisioning and configuring virtual machines  
- Deploying **Ansible Automation Platform 2.5 (AAP)**  

This collection is designed for **lab, testing, and learning scenarios** where you want a repeatable environment setup.

---

## 📦 Installation

```bash
### From local tarball
ansible-galaxy collection install homelab-mylab-1.0.0.tar.gz -f

## After installation, playbooks and roles will be available under:
~/.ansible/collections/ansible_collections/homelab/mylab/
```
## Collection Layout

```text
playbooks/
├── site.yml           # Main entrypoint  
├── deployvms.yml      # VM provisioning  
├── aap25.yml          # AAP deployment  
└── inventories/  
    └── hosts.yml      # Example inventory  

roles/
├── deploy_vms/        # Role to provision virtual machines  
├── aap25_deploy/      # Role to install Ansible Automation Platform 2.5  
```

## Prerequisites
```text
Red Hat Satellite Server available with the required subscription and the following repositories synchronized and made available in a content view. AAP Example.

Red Hat Ansible Automation Platform 2.5 for RHEL 9 x86_64
Red Hat Enterprise Linux 9 for x86_64 - AppStream (RPMs)
Red Hat Enterprise Linux 9 for x86_64 - BaseOS (RPMs)

Populate the file: register-satellite.sh in the deployvms role with the host registration call. Example:

set -o pipefail && curl -sS --insecure 'https://satellite.lab.example/register?activation_keys=AK_RHEL9_PRD&force=true&hostgroup_id=1&location_id=2&organization_id=1&update_packages=false' -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0LCJpYXQiOjE3NTczNDAzODQsImp0aSI6ImNkMDFjMDE5OTBjYTY0YTI0ZDM2MTBlNjM5ODkxZTRlMzJhOGI5MGYwNGQ2NDdiNGVkYjI5MzY3OWRkNDcyYTciLCJzY29wZSI6InJlZ2lzdHJhdGlvbiNnbG9iYWwgcmVnaXN0cmF0aW9uI2hvc3QifQ.ckubPJpN9aJhm-O0az-Grz0U8ok4zKYY3U1yQikUI5o' | bash

Configure your /etc/hosts with servers that will be created
```

### Run only VM provisioning
```bash
ansible-playbook homelab.mylab.site.yml --tags deployvms -e target_env=aap
```
### Run only AAP deployment
```bash
ansible-playbook homelab.mylab.site.yml --tags deployvms,aap25 -e target_env=aap
```

| Tag         | Description                              |
| ----------- | ---------------------------------------- |
| `deployvms` | Provision and configure virtual machines |
| `aap25`     | Deploy Ansible Automation Platform 2.5   |


### Requirements

Ansible >= 2.13  
Python >= 3.9  
SSH access to target hosts or virtualization environment

Author

Romulo Macedo (@rcmacedo)
