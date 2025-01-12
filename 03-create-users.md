# 🚀 **Ansible Playbook: Managing Users, Groups, Package Management**

This project demonstrates how to manage users, groups, chocolatey and updates using Ansible.

## 📂 **Project Structure**

```
/03-create-users
├── users.yml          # Playbook for managing users and authentication
├── packages.yml     # Playbook for managing chocolatey package manager
├── inventory
├── vars/
│   └── users_vars.yml # Variables for users


```

---

### **Login as student into ansible-1 control node:**
```bash
mkdir ~/03-create-users
touch ~/03-create-users/users.yml ~/03-create-users/packages.yml  ~/03-create-users/inventory
mkdir ~/03-create-users/vars
touch ~/03-create-users/vars/users_vars.yml 
tree ~/03-create-users
cd ~/03-create-users
```

### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[infra]
instance1
instance2


[infra:vars]
ansible_connection=winrm
ansible_winrm_transport=credssp
ansible_winrm_server_cert_validation=ignore
ansible_port=5986
ansible_user=Administrator
ansible_password="devoteam@2024"

```
---

## 🛠️ **Variables for Users and Groups**

Define users and their groups in a variable file.

**File:** `vars/users_vars.yml`

```yaml
---
users:
  - username: user1
    group: webadmin
    password: devoteam@2024

  - username: user2
    group: dbadmin
    password: devoteam@2024

```

---

## 📋 **Playbook: `users.yml`**

This playbook creates users, passwords, manages their group permissions.

**File:** `users.yml`

```yaml
---
- name: Create multiple local users
  hosts: infra
  vars_files:
    - vars/users_vars.yml
  tasks:
    - name: Create a local group
      ansible.windows.win_group:
        name: "{{ item.group }}"
        description: "This is a custom local group"
        state: present
      with_items: "{{ users }}"

    - name: Create a user and add to the group
      ansible.windows.win_user:
        name: "{{ item.username }}"
        password: "{{ item.password }}"
        description: "Created by Ansible"
        groups:
          - "{{ item.group }}"
        state: present
      with_items: "{{ users }}"
      no_log: true


    - name: Add user to Administrators group
      ansible.windows.win_group_membership:
        name: Administrators
        members:
          - "{{ users[0].username }}"
        state: present


    - name: Set folder permissions for a group
      ansible.windows.win_acl:
        path: 'C:\Program Files'
        user: dbadmin
        type: allow
        rights: fullcontrol
        state: present
```

### **Task Explanation:**
1. **Add webadmin group and dbadmin:** Ensures the `webadmin` and `dbadmin` groups exist.
2. **Create user accounts:** Creates users based on variables.
3. **Control group memebership:** Add users to Administrators group .
4. **Modify Folder Permissions:** Grants `dbadmin` group the full control over a folder.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run users.yml -m stdout -i inventory 
```

### 🧪  **Verify:**
```bash
powershell --server instance1 --command 'Get-LocalUser'
powershell --server instance1 --command 'Get-LocalGroup'
```

## 📋 **Playbook: `packages.yml`**
Install multiple packages with specific versions


**File:** `packages.yml`

```yaml
---
- name: Update all packages using Chocolatey
  hosts: infra
  gather_facts: false
  vars:
    choco_packages:
      - name: nodejs
        version: 13.0.0
      - name: python
        version: 3.6.0
  tasks:

  - name: Install specific versions of packages sequentially
    chocolatey.chocolatey.win_chocolatey:
      name: "{{ item.name }}"
      version: "{{ item.version }}"
      force: yes
    loop: "{{ choco_packages }}"


  - name: Check python version
    ansible.windows.win_command: python --version
    register: check_python_version

  - name: Check nodejs version
    ansible.windows.win_command: node --version
    register: check_node_version

  - ansible.builtin.debug:
      msg: Python Version is {{ check_python_version.stdout_lines[0] }} and NodeJS version is {{ check_node_version.stdout_lines[0] }}

  - name: Update all installed packages
    chocolatey.chocolatey.win_chocolatey:
      name: all
      state: latest


  - name: Check python version
    ansible.windows.win_command: python --version
    register: check_python_version

  - name: Check nodejs version
    ansible.windows.win_command: node --version
    register: check_node_version

  - ansible.builtin.debug:
      msg: Python Version is {{ check_python_version.stdout_lines[0] }} and NodeJS version is {{ check_node_version.stdout_lines[0] }}

```

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run packages.yml -m stdout -i inventory 
```


## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
- name: Clean up user accounts and configurations
  hosts: infra
  vars_files:
    - vars/users_vars.yml

  vars:
    choco_packages:
      - nodejs
      - python
  tasks:
  - name: Remove local groups
    ansible.windows.win_group:
      name: "{{ item.group }}"
      state: absent
    with_items: "{{ users }}"

  - name: Remove the users
    ansible.windows.win_user:
      name: "{{ item.username }}"
      state: absent 
    with_items: "{{ users }}"

  - name: Remove packages sequentially
    chocolatey.chocolatey.win_chocolatey:
      name: "{{ item }}"
      state: "absent"
      force: yes
    loop: "{{ choco_packages }}"



```
   ```bash
   ansible-navigator run playbook_clean.yml -i inventory -m stdout 
   ```

