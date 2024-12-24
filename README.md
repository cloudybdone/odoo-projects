# Odoo Setup and Configuration Documentation

## Tasks Overview
1. **Update Debian Server Packages**
2. **Create a Sudo User to Run Odoo Instances and Services** (user: `odoo`)

---

## Python Setup

### Install Python 3.11 for BOFL Project (Odoo Version: 17.00)
```bash
sudo apt update
sudo apt install python3.11
```

---

## Git Setup

### Install Git
```bash
sudo apt install git
```

### Steps:

1. **Create a Folder for the CRM Project**
   ```bash
   mkdir -p /opt/odoo-project/
   cd /opt/odoo-project/
   ```

2. **Generate SSH Key for Cloning CRM Project from GitHub**

   - **Step 1: Check for Existing SSH Keys**
     ```bash
     ls -al ~/.ssh
     ```

   - **Step 2: Generate a New SSH Key**
     ```bash
     ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
     ```

   - **Step 3: Add SSH Key to the SSH Agent**
     ```bash
     eval "$(ssh-agent -s)"
     ssh-add ~/.ssh/id_rsa
     ```

   - **Step 4: Copy the SSH Key to Your Clipboard**
     ```bash
     cat ~/.ssh/id_rsa.pub
     ```

   - **Step 5: Add SSH Key to GitHub Account**
     - Log into GitHub and navigate to **Settings > SSH and GPG keys > New SSH key**.
     - Paste the copied key.

   - **Step 6: Test SSH Connection**
     ```bash
     ssh -T git@github.com
     ```

3. **Clone the CRM Project from GitHub**
   ```bash
   git clone git@github.com:technohaven-company-limited/technohaven_automation.git
   ```

---

## PostgreSQL Setup

### Install PostgreSQL (Latest Version)
```bash
sudo apt -y install postgresql
```

### Steps:
1. **Verify PostgreSQL Status**
   ```bash
   sudo systemctl status postgresql
   sudo systemctl start postgresql
   ```

2. **Create a Database for Testing**
   ```bash
   sudo su postgres
   psql
   CREATE ROLE crm WITH LOGIN CREATEDB CREATEROLE SUPERUSER PASSWORD 'tc123';
   ```

3. **Allow PostgreSQL Port on Firewall**
   ```bash
   sudo apt-get install ufw
   sudo ufw enable
   sudo ufw allow 5432/tcp
   sudo ufw reload
   ```

4. **Check PostgreSQL Logs**
   ```bash
   sudo tail -f /var/log/postgresql/postgresql-<version>-main.log
   ```

---

## Virtual Environment Setup

1. **Install Python Virtual Environment**
   ```bash
   sudo apt install python3.11-venv
   ```

2. **Setup Virtual Environment**
   ```bash
   python3.11 -m venv venv
   source venv/bin/activate
   pip install wheel
   python3.11 -m pip install -r requirements.txt
   ```

3. **Resolve Errors (if any):**

   - **For psycopg2 Error:**
     ```bash
     sudo apt-get update
     sudo apt-get install -y build-essential libpq-dev python3-dev
     pip install psycopg2-binary
     pip install psycopg2
     ```

   - **For python-ldap Error:**
     ```bash
     sudo apt-get install -y build-essential python3-dev libldap2-dev libsasl2-dev libssl-dev
     pip install --upgrade pip setuptools wheel
     pip install python-ldap --verbose
     pip install ldap3
     ```

4. **Verify Installed Dependencies**
   ```bash
   pip list
   ```

---

## Configuration File Setup

### Create a Configuration File
- File Path: `/etc/crm_tcl.conf`

```ini
[options]
pg_dump = /usr/pgsql-15/bin/pg_dump
db_host=localhost
db_port=5432
db_password=tcl123
admin_passwd=tcl@123
db_user=crm
#db_name = crm_db
xmlrpc_port = 8071
secure_xmlrpc_port = 8091
addons_path=/opt/odoo-project/crm/odoo/addons,/opt/odoo-project/crm/odoo/custom
data_dir=/opt/odoo-project/crm/odoo/data_dir
logfile=/var/log/odoo/crm_tcl.log
log_level=debug
proxy_mode=True
http_enable=True
https_enable=True
http_port=8071
https_port=443
secure_pkey_file=/etc/ssl/private/odoo71.key
secure_cert_file=/etc/ssl/certs/odoo71.crt
secure_ciphers=TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384
secure_ssl_redirect=True

[resource_settings]
limit_memory_hard=1677721600
limit_memory_soft=629145600
limit_request=8192
limit_time_cpu=600
limit_time_real=1200
max_cron_threads=1
workers=60
```

---

## Service File Setup

### Create a Service File
- File Path: `/etc/systemd/system/crm_tcl.service`

```ini
[Unit]
Description=Odoo
Documentation=http://www.odoo.com
Wants=postgresql-15.service
After=network-online.target postgresql-15.service

[Service]
Type=simple
SyslogIdentifier=odoo
PermissionsStartOnly=true
User=odoo
Group=postgres
ExecStart=/opt/odoo-project/crm/odoo/venv/bin/python3.11 /opt/odoo-project/crm/odoo/odoo-bin -c /etc/crm_tcl.conf
StandardOutput=journal+console
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

### Allow Instance Port on Firewall
```bash
sudo ufw allow 8071/tcp
sudo ufw reload
```

---

## Database Backup and Restore

### Backup the Current Database
```bash
pg_dump -U crm -h localhost -p 5432 crm_db > dump_crm_db_24Dec.sql
```

### Restore the Database
1. **Create a New Database:**
   ```bash
   createdb -O owner_name new_db_name
   ```

2. **Restore Database:**
   ```bash
   psql new_db_name < dump_crm_db_24Dec.sql
   
