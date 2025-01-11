# 🚀 **Ansible Playbooks: Simple File Operations with Copy and Lineinfile Modules**

This project contains three simple Ansible playbooks demonstrating the use of the `copy` and `lineinfile` modules.

## 📂 **Project Structure**

```
/01-simple-copy
├── playbook1.yml   # Write a simple line to a file
├── playbook2.yml   # Write a line with system facts
├── playbook3.yml   # Replace a specific string in a file
├── inventory       # Inventory file for target hosts
```

---
### **Login as student into ansible-1 control node:**
```bash
mkdir ~/01-simple-copy
touch ~/01-simple-copy/playbook1.yml ~/01-simple-copy/playbook2.yml ~/01-simple-copy/playbook3.yml ~/01-simple-copy/inventory
tree ~/01-simple-copy
cd ~/01-simple-copy
```

### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[group1]
instance1
instance2

[group2]
instance2

[all:vars]
ansible_connection=winrm
ansible_winrm_transport=credssp
ansible_winrm_server_cert_validation=ignore
ansible_port=5986
ansible_user=Administrator
ansible_password="devoteam@2024"



```

## 🛠️ **1. Playbook 1: Write a Simple Line to a File**

**Playbook:** `playbook1.yml`

```yaml
---
- name: Write a simple line to a file
  hosts: group1 #only hosts of GROUP "group1"

  tasks:
    - name: Write a static line to a file
      ansible.builtin.win_copy:
        dest: C:\Users\Administrator\AppData\Local\Temp\simple_file.txt
        content: "This is a static line written by Ansible."

```

### **Explanation:**
- Creates a file `C:\Users\Administrator\AppData\Local\Temp\simple_file.txt`.
- Writes a static line to the file.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run  playbook1.yml -m stdout -i inventory
```
### 🚦 **Verify:**
```bash
powershell --server instance1 --command 'Get-Content -Path C:\Users\Administrator\AppData\Local\Temp\simple_file.txt'
powershell --server instance2 --command 'Get-Content -Path C:\Users\Administrator\AppData\Local\Temp\simple_file.txt'
```
---

## 🧪 **2. Playbook 2: Write a Line with System Facts**

**Playbook:** `playbook2.yml`

```yaml
---
- name: Write system facts to a file
  hosts: group2 #only hosts of GROUP "group2"
  tasks:
    - name: Write Memory and IP information to a file
      ansible.builtin.win_copy:
        dest: C:\Users\Administrator\AppData\Local\Temp\facts_file.txt
        content: |
          Memory: {{ ansible_memtotal_mb }} MB
          IP Address: {{ ansible_facts['interfaces'][0]['ipv4']['address'] }}


```

### **Explanation:**
- Gathers system facts (`ansible_memtotal_mb`, `ansible_facts['interfaces'][0]['ipv4']['address']`).
- Writes Memory, and IP Address information to `C:\Users\Administrator\AppData\Local\Temp\facts_file.txt`.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run playbook2.yml -m stdout -i inventory 
```
### 🚦 **Verify:**
```bash
powershell --server instance2 --command 'Get-Content -Path C:\Users\Administrator\AppData\Local\Temp\facts_file.txt'
```
---

## ✏️ **3. Playbook 3: Replace a Specific String in a File**

**Playbook:** `playbook3.yml`

```yaml
---
- name: Replace a specific string in a file
  hosts: all,!instance2
  tasks:
    - name: Replace 'static' with 'dynamic' in the file
      ansible.builtin.win_lineinfile:
        path: C:\Users\Administrator\AppData\Local\Temp\simple_file.txt
        regexp: 'static'
        line: 'This is a dynamic line written by Ansible.'
```

### **Explanation:**
- Searches for the word `static` in `C:\Users\Administrator\AppData\Local\Temp\simple_file.txt`.
- Replaces it with the word `dynamic`.

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run playbook3.yml -m stdout -i inventory 
```
### 🚦**Verify:**
```bash
powershell --server instance1 --command 'Get-Content -Path C:\Users\Administrator\AppData\Local\Temp\simple_file.txt'
```

---

## ✅ **Expected Results**

1. **Playbook 1:** Creates `C:\Users\Administrator\AppData\Local\Temp\simple_file.txt` with a static line.
2. **Playbook 2:** Creates `C:\Users\Administrator\AppData\Local\Temp\facts_file.txt` with system facts.
3. **Playbook 3:** Updates `C:\Users\Administrator\AppData\Local\Temp\simple_file.txt`, replacing `static` with `dynamic`.

---

## 📖 **Key Concepts Used**

- **Copy Module:** Write static or dynamic content to files.
- **System Facts:** Retrieve CPU, memory, and IP information.
- **Lineinfile Module:** Replace or modify specific lines in files.

---

## 🚦 **4. Clean Up the Environment**

After testing, clean up the environment:
**Playbook:** `clean.yml`
```yaml
---
- name: Clean nodes
  hosts: all
  tasks:
    - name: Remove previously created files
      ansible.builtin.win_file:
        path: "{{ item }}"
        state: absent
      loop:
      - C:\Users\Administrator\AppData\Local\Temp\simple_file.txt
      - C:\Users\Administrator\AppData\Local\Temp\facts_file.txt
```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run clean.yml -m stdout -i inventory 
```
---

### 🚦 **Verify:**
```bash
powershell --server instance1 --command 'Get-ChildItem -Path "C:\Users\Administrator\AppData\Local\Temp\" -File'

```
### **Explanation:**
- **C:\Users\Administrator\AppData\Local\Temp\simple_file.txt** and **C:\Users\Administrator\AppData\Local\Temp\facts_file.txt** should not be exist anymore.
