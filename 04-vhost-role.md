# 🚀 **Ansible Role: vhost_role - Apache Virtual Host Deployment**

This project contains an Ansible role named `vhost_role` to automate the deployment, configuration, and validation of an Apache web server with a virtual host.

## 📂 **Project Structure**

```
/04-vhost-role
├── call-vhost-role.yml   # Playbook using the vhost_role role
├── inventory
├── roles/
│   ├── vhost_role/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   ├── html/
│   │   │   │   └── index.html
```
### **Login as student into ansible-1 control node:**
```bash
mkdir ~/04-vhost-role
touch ~/04-vhost-role/call-vhost-role.yml  ~/04-vhost-role/inventory
mkdir ~/04-vhost-role/roles
ansible-galaxy init vhost_role --init-path ~/04-vhost-role/roles

touch ~/04-vhost-role/roles/vhost_role/templates/vhost.conf.j2 
touch ~/04-vhost-role/roles/vhost_role/files/index.html
tree ~/04-vhost-role
cd ~/04-vhost-role
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[webservers]
instance1
instance2

[webservers:vars]
ansible_connection=winrm
ansible_winrm_transport=credssp
ansible_winrm_server_cert_validation=ignore
ansible_port=5986
ansible_user=Administrator
ansible_password="devoteam@2024"


```

---

## 🛠️ **1. roles/vhost_role**

### **Description:**  
This role installs and configures the Apache HTTP server with a virtual host.


**Template File:** `roles/vhost_role/templates/vhost.conf.j2`

```apache
# {{ ansible_managed }}

<VirtualHost *:{{ http_port }}>
    ServerAdmin webmaster@{{ ansible_fqdn }}
    ServerName {{ ansible_fqdn }}
    ErrorLog "C:/Apache24/logs/{{ ansible_hostname }}-error.log"
    CustomLog "C:/Apache24/logs/{{ ansible_hostname }}-common.log" common
    DocumentRoot "C:/Apache24/htdocs/"

    <Directory "C:/Apache24/htdocs/">
        Options +Indexes +FollowSymLinks +Includes
        Require all granted
    </Directory>
</VirtualHost>

```

**Files:** `roles/vhost_role/files/index.html`

```
Welcome to Ansible Roles!

    This page is deployed and managed using Ansible.
    Happy Automation 🚀✨
```

**Vars File:** `roles/vhost_role/defaults/main.yml`

```yaml
---
http_port: 8085

```


**Tasks File:** `roles/vhost_role/tasks/main.yml`

```yaml
---
- name: Download Apache HTTP Server
  win_get_url:
    url: 'https://www.apachelounge.com/download/VS17/binaries/httpd-2.4.62-240904-win64-VS17.zip'
    dest: 'C:\Users\Administrator\AppData\Local\httpd-2.4.54-win64-VS16.zip'

- name: Extract Apache HTTP Server
  win_unzip:
    src: 'C:\Users\Administrator\AppData\Local\httpd-2.4.54-win64-VS16.zip'
    dest: 'C:\'
    creates: 'C:\Apache24\bin\httpd.exe'

- name: Configure Apache ServerRoot
  win_lineinfile:
    path: 'C:\Apache24\conf\httpd.conf'
    regexp: '^ServerRoot'
    line: 'ServerRoot "C:/Apache24"'
  notify: Restart Apache service

- name: Configure Apache Listen Port
  win_lineinfile:
    path: 'C:\Apache24\conf\httpd.conf'
    regexp: '^Listen'
    line: 'Listen {{ http_port }}'
  notify: Restart Apache service

- name: Configure Apache Include option
  win_lineinfile:
    path: 'C:\Apache24\conf\httpd.conf'
    line: 'Include conf/extra\vhost.conf'
  notify: Restart Apache service

# Deploy the virtual host configuration
- name: vhost file is rendered
  win_template:
    src: vhost.conf.j2
    dest: 'C:\Apache24\conf\extra\vhost.conf'
  notify:
    - Restart Apache service

- name: Install Apache as a Windows service
  win_shell: |
    C:\Apache24\bin\httpd.exe -k install -n my_apache
  args:
    executable: cmd
  register: install
  failed_when: 
    - install.rc != 0 
    - "'Service is already installed' not in install.stderr"

# Deploy a custom index.html file to the Apache root directory
- name: Deploy custom index.html
  win_copy:
    src: files/index.html
    dest: 'C:\Apache24\htdocs\index.html'


- name: Open HTTP port in Windows Firewall
  win_firewall_rule:
    name: 'Allow HTTP'
    enable: yes
    localport: "{{ http_port }}"
    protocol: tcp
    action: allow


    
```

**Handlers File:** `roles/vhost_role/handlers/main.yml`

```yaml
---
# Restart Apache service when configuration changes
- name: Restart Apache service
  win_service:
    name: 'my_apache'
    state: restarted
    start_mode: auto
```

---

## 📋 **2.1 call-vhost-role.yml (Beginner) **

### **Description:**  
This playbook uses the `vhost_role` role to configure and deploy the Apache virtual host and ensures web content is copied.

**Master playbook:**   `call-vhost-role.yml`

```yaml
---
- name: Use vhost_role role playbook
  hosts: webservers
  tasks:
  - name: Include vhost role
    ansible.builtin.include_role:
      name: vhost_role

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run call-vhost-role.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://instance1:8085
curl http://instance2:8085

```

## 📋 **2.2 call-vhost-role.yml (Advance) **

### **Description:**  
This playbook uses the `vhost_role` role to configure and deploy the Apache virtual host and ensures web content is copied.

**Master playbook:**   `call-vhost-role.yml`

```yaml
---
- name: Use vhost_role role playbook
  hosts: webservers
  pre_tasks:
    - name: pre_tasks message
      ansible.builtin.debug:
        msg: 'Ensure web server configuration.'
  tasks:
    - name: tasks message
      ansible.builtin.debug:
        msg: 'This is the tasks order.' 
  roles:
    - vhost_role

  post_tasks:
    - name: post_tasks message
      ansible.builtin.debug:
        msg: 'Web server is configured.'
```
---
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run call-vhost-role.yml -m stdout -i inventory -e http_port=8090
```

### 🚦  **Verify:**
```bash
curl http://instance1:8090
curl http://instance2:8090

```
**Order of execution:**
- pre tasks
- roles
- tasks
- handlers notified
- post tasks
---

## 📖 **Key Concepts Used**

- **Roles:** Structure tasks, handlers, files, and templates.
- **Variables:** Dynamic configurations in templates.
- **Handlers:** Restart services on configuration changes.
- **Templates:** Jinja2 for virtual host configurations.
- **File Management:** Copy static content.

---

