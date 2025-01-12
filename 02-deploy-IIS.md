---
- name: Install theIIS web service
  hosts: windows

  tasks:
    - name: InstallIIS
      ansible.windows.win_feature:
        name: Web-Server
        state: present

    - name: StartIIS service
      ansible.windows.win_service:
        name: W3Svc
        state: started

    - name: Create website index.html
      ansible.windows.win_copy:
        content: "{{iis_test_message }}"
        dest: C:\Inetpub\wwwroot\index.html

    - name: Show website address
      ansible.builtin.debug:
        msg: http://{{ ansible_host }}

# 🚀 **Ansible Playbook: Web Server Deployment and Testing**

This project contains two Ansible playbooks to automate the deployment, configuration, and validation of IIS web servers.

## 📂 **Project Structure**

```
/02-deploy-IIS
├── deploy_IIS.yml      # Playbook for deploying and configuring web servers
├── get_web_content.yml # Playbook for testing web server content

```
### **Login as student into ansible-1 control node:**
```bash
mkdir ~/02-deploy-IIS
touch ~/02-deploy-IIS/deploy_iis.yml   ~/02-deploy-IIS/get_web_content.yml ~/02-deploy-IIS/site.yml ~/02-deploy-IIS/inventory
tree ~/02-deploy-IIS
cd ~/02-deploy-IIS
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


## 🛠️ ** deploy_iis.yml (Beginner) ** 

### **Description:**  
This playbook installs and configures IIS servers on managed hosts.

```yaml
---
- name: Install theIIS web service
  hosts: webservers
  vars:
   iis_port: 6060
   iis_test_message: "This IIS is managed by Ansible"
  tasks:
    - name: Install IIS
      ansible.windows.win_feature:
        name: Web-Server
        state: present

    - name: Start IIS service
      ansible.windows.win_service:
        name: W3Svc
        state: started

    - name: Create website index.html
      ansible.windows.win_copy:
        content: "{{iis_test_message }}"
        dest: C:\Inetpub\wwwroot\index.html

    - name: Show website address
      ansible.builtin.debug:
        msg: http://{{ ansible_host }}:{{ iis_port }}


    - name: Change IIS binding port
      ansible.windows.win_powershell:
        script: |
        Import-Module WebAdministration
        Set-ItemProperty -Path "IIS:\Sites\Default Web Site" -Name bindings -Value @{protocol="http"; bindingInformation="*:{{ iis_port }}:"}



    - name: Restart IIS service
      win_service:
        name: 'W3Svc'
        state: restarted
        start_mode: auto

    - name: Open HTTP port in Windows Firewall
      win_firewall_rule:
        name: 'Allow HTTP'
        enable: yes
        localport: "{{iis_port }}"
        protocol: tcp
        action: allow

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_iis.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://instance1:6060
curl http://instance2:6060

```

## 🛠️ ** deploy_iis.yml ** 

### **Description:**  
This playbook installs and configures IIS servers on managed hosts.

```yaml
---
- name: Install theIIS web service
  hosts: webservers
  vars:
    iis_sites:
      - name: 'IIS Site 1'
        port: '8081'
        path: 'C:\sites\site1'
        iis_test_message: 'Hello From Site #1'
      - name: 'Ansible Site 2'
        port: '8082'
        path: 'C:\sites\site2'
        iis_test_message: 'Hello From Site #2'


  tasks:
    - name: Install IIS
      ansible.windows.win_feature:
        name: Web-Server
        state: present

    - name: Ensure IIS folders are ready
      ansible.windows.win_file:
        path: "{{ item.path }}"
        state: directory
      with_items: "{{ iis_sites }}"

    - name: Create IIS site
      community.windows.win_iis_website:
        name: "{{ item.name }}"
        state: started
        port: "{{ item.port }}"
        physical_path: "{{ item.path }}"
      with_items: "{{ iis_sites }}"
      notify: restart iis service

    - name: Template simple web site to iis_site_path as index.html
      ansible.windows.win_copy:
        content: '{{ item.iis_test_message }}'
        dest: '{{ item.path }}\index.html'
      with_items: "{{ iis_sites }}"

    - name: Open port for site on the firewall
      community.windows.win_firewall_rule:
        name: "iisport{{ item.port }}"
        enable: true
        state: present
        localport: "{{ item.port }}"
        action: Allow
        direction: In
        protocol: Tcp
      with_items: "{{ iis_sites }}"


  handlers:
    - name: restart iis service
      ansible.windows.win_service:
        name: W3Svc
        state: restarted
        start_mode: auto

```
### 🚦 **Run the Playbook:**
```bash
ansible-navigator run deploy_iis.yml -m stdout -i inventory 
```

### 🚦  **Verify:**
```bash
curl http://instance1:8081
curl http://instance2:8081
curl http://instance1:8082
curl http://instance2:8082

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
        url: http://instance1:6060
        return_content: true
      register: content

    # Print the returned content
    - name: Print the web content
      ansible.builtin.debug:
        msg: "{{ content }}"

    - name: Retrieve web content from instance2
      ansible.builtin.uri:
        url: http://instance2:6060
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


## 📋 **5. site.yml (advance)**

### **Description:**  
This master playbook imports and executes `deploy_iis.yml` and `get_web_content.yml` sequentially.

```yaml
---
# Deploy and configure web servers
- name: Deploy web servers
  ansible.builtin.import_playbook: deploy_iis.yml

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
- name: Install and configure IIS HTTP Server on Windows
  hosts: webservers
  tasks:

    - name: Stop IIS service
      ansible.windows.win_service:
        name: W3Svc
        state: stopped

    - name: Uninstall IIS
      ansible.windows.win_feature:
        name: Web-Server
        state: absent





```
## 🚦 **Run the Playbooks**
   ```bash
   ansible-navigator run  playbook_clean.yml -i inventory -m stdout 
   ```



