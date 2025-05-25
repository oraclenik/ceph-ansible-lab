# Ceph Cluster Installation with Ansible (Ubuntu 24.04 + KVM/VMware)

This repository contains Ansible playbooks to **automate deployment of a Ceph storage cluster** using lightweight Ubuntu 24.04 VMs. It includes Ceph bootstrapping, OSD provisioning, SSH access setup, and container runtime installation — all driven by a single master playbook.

---

## 🖥️ Cluster Layout

```ini
[master]
node-master ansible_host=192.168.178.73

[workers]
node-01 ansible_host=192.168.178.74
node-02 ansible_host=192.168.178.75
node-03 ansible_host=192.168.178.76

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/oracle/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

---

## 📦 Playbooks Overview

| File                                         | Purpose                                                             |
|----------------------------------------------|----------------------------------------------------------------------|
| `ceph-site.yml`                              | **Master playbook** to run all steps in order                       |
| `ceph-prereqs-workers.yml`                   | Prepares worker nodes: wipes `/dev/sdb`, installs base packages     |
| `ceph-bootstrap-master.yml`                  | Bootstraps the cluster using a local patched `cephadm` script       |
| `ceph-generate-and-distribute-ssh-key.yml`   | Generates Ceph’s SSH key and deploys it to worker nodes             |
| `add-ssh-keys.yml`                           | Adds both Node-Master and Ceph Orchestrator SSH keys to workers     |
| `ceph-add-hosts.yml`                         | Adds worker nodes to the Ceph orchestrator                          |
| `ceph-add-osds.yml`                          | Provisions `/dev/sdb` as an OSD on each worker                      |

---

## 🚀 Installation Steps

To run the full deployment sequence, use the master playbook:

```bash
ansible-playbook -i hosts.ini ceph-site.yml
```

Or run playbooks individually:

```bash
ansible-playbook -i hosts.ini ceph-bootstrap-master.yml
ansible-playbook -i hosts.ini ceph-add-osds.yml
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

Access: `https://<node-master-ip>:8443`

Default credentials:
- **Username:** `admin`
- **Password:** `admin` (or set in vars)

---

## 🐞 Known Issues

### ❗ `CEPHADM_REFRESH_FAILED` or `CEPHADM_STRAY_HOST`

Run:

```bash
sudo cephadm shell -- ceph orch host refresh --all
```

Adopt unmanaged daemons (if needed):

```bash
sudo cephadm shell -- ceph orch adopt --name <daemon-name>
```

---

### ❗ `ValueError: too many values to unpack` in cephadm

This is due to AppArmor detection failing. Patch your cephadm with:

```python
parts = line.split(' ')
item = parts[0]
mode = parts[1] if len(parts) > 1 else 'UNKNOWN'
```

✅ **Note**: This repo includes a patched `cephadm` script **only as an example**. You should always patch and verify your version manually against the upstream Ceph release.

---

## ✅ What’s Next?

- Optional: add Kubernetes/Rook integration
- Automate dashboard user/password setup
- Add backup/monitoring integrations

---

> Ideal for homelabs, POCs, learning Ceph orchestration, or reproducible lightweight deployments.
