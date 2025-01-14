# 🚀 **Ansible Playbook: Managing sensitive data**


This project includes an Ansible playbook designed to create users, securely handle user passwords as sensitive data, and encrypt them.

## 📂 **Project Structure**

```
/07-managing-senistive-data
├── create_users.yml      # Playbook for creating users
├── secret.yml            # Vault encrypted data
├── key.txt               # Vault encryption key
├── inventory
```
### **Login as student into ansible-1 control instance:**
```bash
mkdir ~/07-managing-senistive-data
touch ~/07-managing-senistive-data/create_users.yml ~/07-managing-senistive-data/key.txt  ~/07-managing-senistive-data/inventory
tree ~/07-managing-senistive-data
cd ~/07-managing-senistive-data
```

### **Ensure your hosts are defined in the inventory file:**


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
## 🛠️ ** secret.yml ** 

### **Description:**  
This file conatins the encrypted user password

```bash
ansible-vault create secret.yml
New Vault password: devoteam
Confirm New Vault password: devoteam

```
use your desired encryption key

declare username and password variables
**--(press 'a' character inside the editor to edit)--**
```txt
username: myvault_user
password: devoteam@2024
```
save thefile
**--(press ESC and then :wq) to save and exit--**

## 🛠️ ** create_users.yml ** 

### **Description:**  
This playbook creates system users and ensure that password authentication is enabled

```yaml
---
- name: Create user accounts 
  hosts: infra 
  vars_files:
    - secret.yml
  tasks:
    - name: Create a user 
      ansible.windows.win_user:
        name: "{{ username }}"
        password: "{{ password }}"
        description: "Created by Ansible"
        state: present

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run create_users.yml --pae false -m stdout -i inventory --vault-id @prompt
```


## 🛠️ ** key.txt ** 

### **Description:**  
This file conatins the encryption key as plain text

```txt
devoteam
```
### 🚦 **Run the Playbook with key file instead of prompts:**
```bash
ansible-navigator run create_users.yml -m stdout -i inventory --vault-password-file=key.txt
```



### 🚦  **Verify:**
Ensure you can login to the instances with myvault_user
```bash
powershell --server instance1 --command 'Get-LocalUser'
powershell --server instance2 --command 'Get-LocalUser'


```
