# 🚀 **Ansible Playbook: Using Roles and Modules from Content Collections**

This project demonstrates how to install Ansible Content Collections, use roles, and leverage modules from those collections.

## 📂 **Project Structure**

```
/06-collections
├── backup.yml             # Playbook using my_namespace.my_collection  collection
├── inventory
├── collections/
│   ├── my_namespace/my_collection/
```
### **Login as student into ansible-1 control instance:**
```bash
mkdir ~/06-collections
touch ~/06-collections/backup.yml  ~/06-collections/inventory
mkdir ~/06-collections/collections

tree ~/06-collections
cd ~/06-collections
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[infra]
instance1
instance2

[all:vars]
ansible_connection=winrm
ansible_winrm_transport=credssp
ansible_winrm_server_cert_validation=ignore
ansible_port=5986
ansible_user=Administrator
ansible_password="devoteam@2024"


```
---

## 🛠️ **1. Install Ansible Collections**

### **Install my_namespace.my_collection  Collection Directly**

Use the `ansible-galaxy` command to install the `my_namespace.my_collection ` collection directly from a tarball.

```bash
ls ~/artifacts/my_namespace-my_collection-1.0.0.tar.gz
ansible-galaxy collection install ~/artifacts/my_namespace-my_collection-1.0.0.tar.gz -p collections/
```

### **Verify Installed Collections**

```bash
ansible-galaxy collection list -p collections
```

**Expected Output:**
```
Collection               Version
------------------------ -------
my_namespace.my_collection               1.0.0
```

---

## 📋 **2. Backup Configuration Using my_namespace.my_collection  Collection**

**Playbook:** `backup.yml`

```yaml
---
- name: Backup the system configuration
  hosts: all
  tasks:
    # Backup files with my_namespace.my_collection.backup role
    - name: Ensure files are saved
      ansible.builtin.include_role:
        name: my_namespace.my_collection.backup
```


### **Run the Playbook**
```bash
ansible-navigator run backup.yml -i inventory -m stdout -e backup_id=MyID ## Should fail

ansible-navigator run backup.yml -i inventory -m stdout -e backup_id=my_id

```
### 🚦  **Verify: Ensure the Backup**
Execute the following command to verify that the backup was already taken.

```bash

powershell --server instance1 --command 'Get-ChildItem -Path C:\mybackup_dir\Ansible_backup-my_id'
powershell --server instance2 --command 'Get-ChildItem -Path C:\mybackup_dir\Ansible_backup-my_id'

```
**Expected response:**
Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        1/14/2025   8:52 AM                Program Files                                                         
d-----        1/14/2025   8:52 AM                Program Files (x86) 
---

## 📖 **Key Concepts Used**

- **Ansible Galaxy:** Install and manage collections with `ansible-galaxy`.
- **Modules:** Use `my_namespace.my_collection.check_capital_letter` to verify inputs.
- **Roles:** Utilize `my_namespace.my_collection.backup` 


---




