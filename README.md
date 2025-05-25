# Ceph Cluster Installation with Ansible (Ubuntu 24.04 + KVM/VMware)

This repository contains Ansible playbooks to **provision a Ceph storage cluster** using lightweight VMs running Ubuntu 24.04. The cluster consists of 1 master node and 3 worker nodes. Ceph is installed using `cephadm`, OSDs are provisioned from `/dev/sdb`, and SSH + Docker + networking is handled end-to-end with Ansible.

---

## 🖥️ Cluster Layout

```ini
[master]
node-master ansible_host=192.168.1.100

[workers]
node-01 ansible_host=192.168.1.101
node-02 ansible_host=192.168.1.102
node-03 ansible_host=192.168.1.103

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/user/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

---

## ⚙️ Playbooks Overview

| File                                 | Purpose                                                             |
|--------------------------------------|----------------------------------------------------------------------|
| `ceph-site.yml`                     | **Master playbook** to run all steps in order                       |
| `ceph-prereqs.yml`                  | Installs base packages, wipes `/dev/sdb`, ensures `chrony` running  |
| `ceph-install-docker.yml`           | Installs Docker on all worker nodes (required for cephadm)         |
| `ceph-bootstrap.yml`                | Bootstraps the cluster using a local patched cephadm script         |
| `ceph-generate-and-distribute-ssh-key.yml` | Generates Ceph's SSH key and deploys it to worker nodes     |
| `add-node-master-key.yml`           | Adds master SSH key to workers for test access                      |
| `ceph-add-hosts.yml`                | Adds worker nodes to the Ceph orchestrator                         |
| `ceph-add-osds.yml`                 | Provisions `/dev/sdb` as an OSD on each worker                     |

---

## 🧱 Installation Steps

To run the full deployment sequence, use the master playbook:

```bash
ansible-playbook -i hosts.ini ceph-site.yml
```

You can also run individual playbooks manually as needed:

```bash
ansible-playbook -i hosts.ini ceph-prereqs.yml
ansible-playbook -i hosts.ini ceph-bootstrap.yml
# etc...
```

---

## 🧪 Verification

### Ceph Status

```bash
sudo cephadm shell -- ceph -s
```

### OSD Tree

```bash
sudo cephadm shell -- ceph osd tree
```

### Dashboard

Access at: `https://<master-ip>:8443`

Default creds set during bootstrap:
- Username: `admin`
- Password: `admin`

---

## 🐞 Known Issues + Fixes

### ❌ AppArmor Error: `ValueError: too many values to unpack`

**Fix**: Patch `/var/lib/ceph/<fsid>/cephadm.*` on **all nodes** (master + workers):

Replace:

```python
item, mode = line.split(' ')
```

With:

```python
parts = line.split(' ')
item = parts[0]
mode = parts[1] if len(parts) > 1 else 'UNKNOWN'
```

Or disable AppArmor check:

```python
def _fetch_apparmor():
    return []
```

> ☑️ **Note**: The `cephadm` script included in this repo is a patched version just for convenience and reference. It’s recommended to manually patch your `cephadm` from the official source to ensure you use the latest supported release and apply compatibility fixes specific to your OS/version.

---
