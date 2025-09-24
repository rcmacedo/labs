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
