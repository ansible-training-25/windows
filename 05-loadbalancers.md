# 🚀 **Ansible Playbook: Apache Application Deployment and Nginx Load Balancer**

This project contains Ansible playbooks to automate the deployment, configuration, and validation of Proxy and webserver setup.



## 📂 **Project Structure**

```
/05-loadbalancers
├── site.yml             # Master playbook to run both roles
├── inventory
├── roles/
│   ├── apache/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   └── vhost.conf.j2
│   │   ├── files/
│   │   │   ├── html/
│   │   │   │   └── index.html
│   └── nginx/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── files/
│       │   └── nginx-service.xml 
│       └── templates/
│           └── nginx.conf.j2
│           └── nginx_vhost.conf.j2

```
### **Login as student into ansible-1 control node:**
```bash
mkdir ~/05-loadbalancers
touch ~/05-loadbalancers/site.yml  ~/05-loadbalancers/inventory
mkdir ~/05-loadbalancers/roles
ansible-galaxy init apache --init-path ~/05-loadbalancers/roles
ansible-galaxy init nginx --init-path ~/05-loadbalancers/roles
touch ~/05-loadbalancers/roles/apache/files/index.html ~/05-loadbalancers/roles/apache/templates/vhost.conf.j2 
touch ~/05-loadbalancers/roles/nginx/templates/nginx.conf.j2  ~/05-loadbalancers/roles/nginx/templates/nginx_vhost.conf.j2

tree ~/05-loadbalancers
cd ~/05-loadbalancers
```
### **Ensure your hosts are defined in the inventroy file:**


**Inventory:** `inventory`
```ini
[loadbalancers]
instance1

[webservers]
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

## 🛠️ **1. roles/apache**

### **Description:**  
This role installs and configures the Apache HTTP server with a virtual host.


**Template File:** `roles/apache/templates/vhost.conf.j2`

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

**Files:** `roles/apache/files/index.html`

```
Welcome to Ansible Apache Backend!

    This page is deployed and managed using Ansible.
    Happy Automation 🚀✨
```

**Vars File:** `roles/apache/defaults/main.yml`

```yaml
---
http_port: 8088

```


**Tasks File:** `roles/apache/tasks/main.yml`

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
    C:\Apache24\bin\httpd.exe -k install -n apache_role_service
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
---
**Handlers File:** `roles/apache/handlers/main.yml`

```yaml
---
# Restart Apache service when configuration changes
- name: Restart Apache service
  win_service:
    name: 'apache_role_service'
    state: restarted
    start_mode: auto
```

---

---

## 🧪 **2. roles/nginx**

### **Description:**  
This role installs and configures Nginx as a load balancer|proxy for Apache servers.

**Template File:** `roles/nginx/templates/nginx.conf.j2`

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    upstream JBoss_backend {
        {% for node in groups['webservers'] %}
        server {{hostvars[node]['ansible_interfaces'][0]['ipv4']['address'] }}:{{ http_port }};
        {% endfor %}
    }

    server {
        listen {{ nginx_port }};

        location / {
            proxy_pass http://JBoss_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}


```
**Vars File:** `roles/nginx/defaults/main.yml`

```yaml
---
nginx_port: 8080

```

**Tasks File:** `roles/nginx/tasks/main.yml`

```yaml
---
# Download NGINX for Windows
- name: Download NGINX
  win_get_url:
    url: 'https://nginx.org/download/nginx-1.25.2.zip'
    dest: 'C:\Users\Administrator\AppData\Local\nginx-1.25.2.zip'

# Extract NGINX
- name: Extract NGINX
  win_unzip:
    src: 'C:\Users\Administrator\AppData\Local\nginx-1.25.2.zip'
    dest: 'C:\'
    creates: 'C:\nginx-1.25.2\nginx.exe'

- name: Render nginx.conf file
  win_template: 
    src: nginx.conf.j2
    dest: 'C:\nginx-1.25.2\conf\nginx.conf'
  notify: Restart NGINX service

- name: Check if the service exists
  win_service:
    name: "nginx"
  register: service_check
  failed_when: false

- block:
  - name: Download NSSM
    win_get_url:
      url: https://nssm.cc/release/nssm-2.24.zip
      dest: C:\nssm.zip

  - name: Extract NSSM
    win_unzip:
      src: C:\nssm.zip
      dest: C:\nssm
      creates: C:\nssm\nssm-2.24\win64\nssm.exe

  - name: Install the service
    win_shell: |
      C:\nssm\nssm-2.24\win64\nssm.exe install nginx C:\nginx-1.25.2\nginx.exe
      C:\nssm\nssm-2.24\win64\nssm.exe set nginx AppDirectory C:\nginx-1.25.2\
      C:\nssm\nssm-2.24\win64\nssm.exe set nginx AppParameters --config C:\nginx-1.25.2\nginx-service.xml
      C:\nssm\nssm-2.24\win64\nssm.exe set nginx AppStdout C:\nginx-1.25.2\service.log
      C:\nssm\nssm-2.24\win64\nssm.exe set nginx AppStderr C:\nginx-1.25.2\service-error.log
    args:
      executable: cmd
  when: not service_check.exists

# Open NGINX port in Windows Firewall
- name: Open NGINX port in Windows Firewall
  win_firewall_rule:
    name: 'Allow NGINX HTTP'
    enable: yes
    localport: "{{ nginx_port }}"
    protocol: tcp
    action: allow







```

**Handlers File:** `roles/nginx/handlers/main.yml`

```yaml
# Restart NGINX service when configuration changes
- name: Restart NGINX service
  win_service:
    name: 'nginx'
    state: restarted
    start_mode: auto

```



## 📋 **3. site.yml**

### **Description:**  
This master playbook imports and executes the `apache` and `nginx` roles sequentially.

```yaml
---
- name: Deploy Apache application servers
  hosts: webservers
  roles:
    - apache

- name: Configure Nginx load balancer
  hosts: loadbalancers
  roles:
    - nginx
          
```

---

### 🚦 **Run the Playbook:**
```bash
ansible-navigator run site.yml -m stdout -i inventory  -e nginx_port=9000 -e http_port=9001
```

### 🚦  **Verify: Ensure the Loadbalanacing**
Execute the following command multiple times to verify that the load balancer is functioning properly with the backend servers.

```bash
curl -v http://instance1:9000

```
**Expected response:** '....nginx . from **instance2**'

```bash
curl http://instance2:9001

```
**Expected response:** '.... apache . . from **node2**'

**Expected Output:**
   - All tasks should complete successfully.
   - Apache servers should be running.
   - Nginx should forward requests to the backend Apache servers.

---

**Try splitting the tasks file of each role**
