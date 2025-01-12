# 🚀 **Ansible Playbook: Web Server Deployment and Testing**

This project contains two Ansible playbooks to automate the deployment, configuration, and validation of web servers.

## 📂 **Project Structure**

```
/02-deploy-webserver
├── deploy_web.yml      # Playbook for deploying and configuring web servers
├── get_web_content.yml # Playbook for testing web server content
└── files/
    └── index.html      # Sample web content file
```
### **Login as student into ansible-1 control node:**
```bash
mkdir ~/02-deploy-webserver
touch ~/02-deploy-webserver/deploy_web.yml   ~/02-deploy-webserver/get_web_content.yml ~/02-deploy-webserver/site.yml ~/02-deploy-webserver/inventory
mkdir ~/02-deploy-webserver/files
touch ~/02-deploy-webserver/files/index.html 
tree ~/02-deploy-webserver
cd ~/02-deploy-webserver
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

---

## 📄 ** files/index.html**

```html
Welcome to Ansible!

    This page is deployed and managed using Ansible.
    Happy Automation 🚀✨

```
---
## 🛠️ ** deploy_web.yml (Beginner)** 

### **Description:**  
This playbook installs and configures Apache HTTP servers on managed hosts.

```yaml
---
- name: Install and configure Apache HTTP Server on Windows
  hosts: webservers
  vars: 
    http_port: 8080
  tasks:
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

    - name: Configure Apache Listen Port
      win_lineinfile:
        path: 'C:\Apache24\conf\httpd.conf'
        regexp: '^Listen'
        line: 'Listen {{ http_port }}'

    - name: Install Apache as a Windows service
      win_shell: |
        C:\Apache24\bin\httpd.exe -k install
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


    - name: Restart Apache service
      win_service:
        name: 'Apache2.4'
        state: restarted
        start_mode: auto

    - name: Open HTTP port in Windows Firewall
      win_firewall_rule:
        name: 'Allow HTTP'
        enable: yes
        localport: "{{ http_port }}"
        protocol: tcp
        action: allow

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_web.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://instance1:8080
curl http://instance2:8080


```

## 🛠️ **1.2. deploy_web.yml (Advance: Handlers)** 

### **Description:**  
This playbook installs and configures Apache HTTP servers on managed hosts.

```yaml
---
- name: Install and configure Apache HTTP Server on Windows
  hosts: webservers
  vars: 
    http_port: 8080
  tasks:
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

    - name: Install Apache as a Windows service
      win_shell: |
        C:\Apache24\bin\httpd.exe -k install
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
    
  handlers:
    - name: Restart Apache service
      win_service:
        name: 'Apache2.4'
        state: restarted
        start_mode: auto

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_web.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://instance1:8080
curl http://instance2:8080


```
---
## 🧪 **4.1. get_web_content.yml (Beginner)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  tasks:
    # Attempt to retrieve web content from target servers
    - name: Retrieve web content from instance1
      ansible.builtin.uri:
        url: http://instance1:8080
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"

    - name: Retrieve web content from instance2
      ansible.builtin.uri:
        url: http://instance2:8080
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"



### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```

## 🧪 **4.2. get_web_content.yml (Intermediate)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  vars:
    target_servers: 
    - instance1
    - instance2
  tasks:
  # Attempt to retrieve web content from target servers
  - name: Retrieve web content
    ansible.builtin.uri:
      url: http://{{ item }}:8080
      return_content: true
    register: content
    loop: "{{ target_servers }}"
    
  # Print the returned content
  - name: Print the web content
    ansible.builtin.debug:
      msg: "{{ content }}"


```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```
---
## 🧪 **4.3. get_web_content.yml (Advance)**

### **Description:**  
This playbook validates the web server's functionality by retrieving its content.

```yaml
---
- name: Test web content
  hosts: localhost
  tasks:

    # Attempt to retrieve web content from target servers
    - name: Retrieve web content and write to error log on failure
      block:
        - name: Retrieve web content
          ansible.builtin.uri:
            url: http://{{ item }}:8080
            return_content: true
          register: content
          loop: "{{ groups['webservers'] }}"

          # Print the returned content
        - name: Print the web content
          ansible.builtin.debug:
            msg: "{{ content }}"

      rescue:
        # Log error to a file if the retrieval fails
        - name: Write to error file
          ansible.builtin.lineinfile:
            path: /home/student/02-deploy-webserver/error.log
            line: "Error retrieving content"
            create: true

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run get_web_content.yml -m stdout -i inventory 
```
---

## 📋 **5. site.yml (advance)**

### **Description:**  
This master playbook imports and executes `deploy_web.yml` and `get_web_content.yml` sequentially.

```yaml
---
# Deploy and configure web servers
- name: Deploy web servers
  ansible.builtin.import_playbook: deploy_web.yml

# Validate the web servers by retrieving web content
- name: Retrieve web content
  ansible.builtin.import_playbook: get_web_content.yml
```

---

## 🚦 **Run the Playbooks**


2. **Run the playbooks using `ansible-navigator`:**
   ```bash
   ansible-navigator run site.yml -i inventory -m stdout 
   ```


---


## 🛠️ ** playbook_clean.yml **

### **Description:**  
This playbook cleans the environment.

```yaml
---


- name: Install and configure Apache HTTP Server on Windows
  hosts: webservers
  tasks:
    - name: Stop Apache service
      win_service:
        name: 'Apache2.4'
        state: stopped
        start_mode: disabled




```
## 🚦 **Run the Playbooks**
   ```bash
   ansible-navigator run  playbook_clean.yml -i inventory -m stdout 
   ```



###please check **force_handlers: yes** at the play level  or **- meta: flush_handlers**   as a task
