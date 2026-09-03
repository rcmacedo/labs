# Ansible Collection: `homelab.mylab`

Collection para provisionar VMs RHEL 9 em KVM/libvirt e instalar o **Ansible Automation Platform (AAP) 2.7** em modo **containerizado e desconectado (disconnected bundle)**, incluindo o **Ansible Lightspeed** com **Automation Intelligent Assistant** (chatbot OpenAI) no mesmo host do Platform Gateway.

Destinada a **laboratório, testes e aprendizado** — não use em produção sem revisar sizing, segurança e requisitos oficiais da Red Hat.

---

## O que é implantado

Topologia **enterprise** (um componente por VM, exceto o gateway):

| VM | Grupo(s) do instalador | Função |
|----|------------------------|--------|
| `aapgateway` | `automationgateway`, `ansiblelightspeed` | Platform Gateway + Lightspeed / Intelligent Assistant |
| `aapcontroller1`, `aapcontroller2` | `automationcontroller` | Automation Controller (HA) |
| `aapexec`, `aapexec2` | `execution_nodes` | Execution nodes |
| `aapdb` | `database` | PostgreSQL |
| `aapeda` | `automationeda` | Event-Driven Ansible |
| `aaphub` | `automationhub` | Automation Hub |
| `aapmetrics` | `automationmetrics` | Metrics service |

O playbook gera o inventário do instalador AAP 2.7 a partir de `roles/deploy_vms/vars/main.yml`. O host de instalação é a sua máquina local (Fedora/RHEL com libvirt).

---

## Pré-requisitos

### No host de instalação (control node)

| Requisito | Detalhe |
|-----------|---------|
| SO | Fedora ou RHEL com **KVM/libvirt** funcional |
| Ansible | **ansible-core 2.14+** (2.16+ se Python ≥ 3.14 no host) |
| Python | ≥ 3.9 |
| Pacotes | `qemu-img`, `virt-install`, `virt-customize`, `virt-resize`, `guestfs-tools`, `python3-pip` |
| Acesso | Membro do grupo `libvirt`, `sudo` para gerenciar VMs e `/etc/hosts` |
| Rede | Rede libvirt (ex.: `external`) com DHCP ou rota para os IPs das VMs |
| SSH | Par de chaves `~/.ssh/id_rsa` (criado automaticamente pelo role se não existir) |

### Licenciamento Red Hat

- Assinatura **RHSM** válida para registrar as VMs RHEL 9 (`registry_user` / `registry_pass` em `vars/main.yml`).
- Download do **AAP 2.7 containerized setup bundle** no [Red Hat Customer Portal](https://access.redhat.com/):

  `ansible-automation-platform-containerized-setup-bundle-2.7-4-x86_64.tar.gz`

### Imagem RHEL 9 para as VMs

- QCOW2 KVM, ex.: `rhel-9.8-x86_64-kvm.qcow2` (caminho configurável em `rhel9_image`).

### Automation Intelligent Assistant (chatbot)

- Chave de API **OpenAI** (ou compatível) para o Lightspeed chatbot.
- Configure em `roles/deploy_vms/vars/secrets.yml` (arquivo local, **não versionado**).

---

## Estrutura da collection

```text
homelab/mylab/
├── galaxy.yml
├── requirements.yml          # dependências Ansible (community.general, community.crypto)
├── playbooks/
│   ├── site.yml              # entrypoint com tags
│   ├── deployvms.yml         # cria VMs + registro RHSM + usuário aap
│   ├── destroyvms.yml        # remove VMs e discos
│   ├── aap27.yml             # instala AAP 2.7
│   └── ocp4.yml              # stub OpenShift (não usado no fluxo AAP)
└── roles/
    ├── deploy_vms/           # KVM, discos, rede, RHSM
    ├── aap27_deploy/         # bundle, inventário, instalador containerizado
    └── ocp4_deploy/          # stub
```

---

## Guia rápido: do clone à instalação do AAP

### 1. Clonar o repositório

```bash
git clone https://github.com/rcmacedo/labs.git
cd labs/homelab
```

### 2. Instalar dependências Ansible

```bash
ansible-galaxy collection install -r mylab/requirements.yml -p ~/.ansible/collections
```

### 3. Configurar variáveis do ambiente

Edite `mylab/roles/deploy_vms/vars/main.yml`:

| Variável | O que ajustar |
|----------|----------------|
| `rhel9_image` | Caminho absoluto da imagem RHEL 9 KVM |
| `vm_storage_path` | Diretório dos discos qcow2 (padrão `/var/lib/libvirt/images`) |
| `domain` | Domínio DNS das VMs (padrão `lab.example`) |
| `network_name` | Nome da rede libvirt - ( padrão external ) |
| `dns_server`, `network_gateway` | DNS e gateway da rede lab |
| `registry_user`, `registry_pass` | Credenciais RHSM |
| `aap_password` | Senha admin dos componentes AAP |
| `vms_all.aap` | IPs, CPU, RAM e tamanho dos discos por VM |

Cada VM aceita:

- `role: automationcontroller` — um único grupo do instalador, ou
- `roles: [automationgateway, ansiblelightspeed]` — vários grupos no mesmo host.

Discos: `disks: [{ size: 60, suffix: root }]` (tamanho em GiB; o role expande a partição root automaticamente).

### 4. Configurar segredos do Intelligent Assistant

```bash
cp mylab/roles/deploy_vms/vars/secrets.yml.example \
   mylab/roles/deploy_vms/vars/secrets.yml
```

Edite `secrets.yml`:

```yaml
lightspeed_chatbot_model_api_key: "sua-chave-openai"
```

Opcional: `lightspeed_chatbot_model_id` em `roles/aap27_deploy/defaults/main.yml` (padrão `gpt-4o-mini`).

> **Nunca** commite `secrets.yml`. Ele já está no `.gitignore` do repositório.

### 5. Baixar e posicionar o bundle AAP 2.7

O tarball **não** é incluído no artefato da collection (arquivo grande). Copie-o para:

```text
mylab/roles/aap27_deploy/files/ansible-automation-platform-containerized-setup-bundle-2.7-4-x86_64.tar.gz
```

### 6. Build e instalação da collection

A partir de `labs/homelab/mylab`:

```bash
ansible-galaxy collection build -f
ansible-galaxy collection install homelab-mylab-1.0.0.tar.gz -f
```

### 7. Executar

Sempre informe o ambiente alvo: `-e target_env=aap`.

**Somente VMs** (primeira vez ou redeploy):

```bash
ansible-playbook homelab.mylab.site --tags deployvms -e target_env=aap --ask-become-pass
```

**Somente AAP 2.7** (VMs já existem):

```bash
ansible-playbook homelab.mylab.site --tags aap27 -e target_env=aap
```

**Fluxo completo** (VMs + AAP):

```bash
ansible-playbook homelab.mylab.site --tags deployvms,aap27 -e target_env=aap --ask-become-pass
```

**Destruir VMs e discos**:

```bash
ansible-playbook homelab.mylab.site --tags destroyvms -e target_env=aap --ask-become-pass
```

### 8. Acompanhar a instalação do AAP

O role dispara o instalador em background. Monitore o log:

```bash
tail -f ~/aap/ansible-automation-platform-containerized-setup-bundle-2.7-4-x86_64/aap_install.log
```

A instalação usa um **venv isolado** em `~/aap/installer-venv` com ansible-core compatível (2.14 ou 2.16, conforme a versão do Python do host).

### 9. Acessar o ambiente

| Serviço | URL |
|---------|-----|
| Platform Gateway | `https://aapgateway.<domain>:443` |
| Usuário admin gateway | `admin` / `aap_password` |
| Lightspeed | Via gateway (grupo `ansiblelightspeed` no mesmo host) |

Substitua `<domain>` pelo valor de `domain` em `vars/main.yml` (ex.: `lab.example`).

---

## Tags do playbook `site.yml`

| Tag | Descrição |
|-----|-----------|
| `deployvms` | Cria VMs, expande discos, registra no RHSM, cria usuário `aap` |
| `destroyvms` | Remove VMs libvirt e discos qcow2 |
| `aap27` | Extrai bundle, gera inventário, executa instalador AAP 2.7 |
| `ocp4` | Stub OpenShift (não relacionado ao fluxo AAP) |

---

## Variáveis importantes do role `aap27_deploy`

| Variável | Padrão | Descrição |
|----------|--------|-----------|
| `aap_setup_tarball` | `ansible-automation-platform-...tar.gz` | Nome do bundle em `files/` |
| `aap_setup_dest` | `~/aap` | Diretório base de trabalho |
| `aap_use_installer_venv` | `true` | Usa venv com ansible-core suportado |
| `aap_lab_skip_ram_check` | `false` no role; `true` em `vars/main.yml` | Ignora preflight de 16 GiB RAM (somente lab) |
| `aap_install_user` | `aap` | Usuário SSH remoto nas VMs (não root) |
| `lightspeed_chatbot_model_id` | `gpt-4o-mini` | Modelo OpenAI do chatbot |

---

## Notas de laboratório

- **`aap_lab_skip_ram_check: true`** — patch no preflight do instalador para VMs com menos de 16 GiB RAM. A instalação pode falhar ou ficar instável sob carga; use apenas em lab.
- **Discos** — mínimo recomendado 60 GiB no controller, gateway (com Lightspeed), hub e execution nodes; 30 GiB em metrics. Ajuste em `disks[].size`.
- **Inventário gerado** — `~/aap/<bundle>/inventory`; não edite manualmente — altere o template `roles/aap27_deploy/templates/inventory.j2` ou as variáveis em `vars/main.yml`.
- **`/etc/hosts` no control node** — o role `aap27_deploy` adiciona entradas para os FQDNs das VMs automaticamente.

---

## Solução de problemas

| Sintoma | Causa provável | Ação |
|---------|----------------|------|
| Push GitHub bloqueado por secret | API key no histórico git | Use `secrets.yml`; nunca commite chaves |
| `no space left on device` no install | Disco VM pequeno | Aumente `disks[].size`, redeploy ou `destroyvms` + `deployvms` |
| `sudo: a password is required` em localhost | Inventário sem grupo `[local]` | Reinstale collection e regenere inventário (`aap27`) |
| `_controller_hostname` undefined | Controller falhou antes do gateway | Verifique `aap_install.log` no controller |
| `virt-customize` / `virt-resize` falha | VM ligada ou permissão libvirt | Pare VMs; execute com `sudo` |
| Instalador rejeita ansible-core 2.18 | Versão do sistema incompatível | Confirme `aap_use_installer_venv: true` |

---

## Requisitos resumidos

```text
ansible-core >= 2.14
Python >= 3.9
collections: community.general, community.crypto
```

---

## Autor

Romulo Macedo ([@rcmacedo](https://github.com/rcmacedo))
