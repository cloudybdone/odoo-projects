# Debian Server Setup for Odoo Project

## Update All Packages on Debian Server
To ensure your Debian server has the latest updates, use the following commands:
```bash
sudo apt update
sudo apt upgrade -y
```

## Create a Sudo User for Running Odoo Instance and Service
Commands to create a sudo user:
```bash
sudo adduser odoo
sudo usermod -aG sudo odoo
```

## Python Setup

### Install Python 3.11
```bash
sudo apt update
sudo apt install python3.11
```

## Git Setup

### Install Git
```bash
sudo apt install git
```

### Create a Folder for the CRM Project
```bash
sudo mkdir -p /opt/odoo-project/
```

### Generate SSH Key for Cloning the CRM Project from GitHub
#### Step 1: Check for Existing SSH Keys
```bash
ls -al ~/.ssh
```
#### Step 2: Generate a New SSH Key
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Options explained:
- `-t rsa`: Specifies RSA key type.
- `-b 4096`: Uses 4096 bits for encryption.
- `-C`: Adds an email comment.

#### Step 3: Add SSH Key to the SSH Agent
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```
#### Step 4: Copy the SSH Key to Your Clipboard
```bash
cat ~/.ssh/id_rsa.pub
```
#### Step 5: Add the SSH Key to Your GitHub Account
1. Log into GitHub.
2. Navigate to **Settings > SSH and GPG keys > New SSH Key**.
3. Paste the copied key and save.

#### Step 6: Test the SSH Connection
```bash
ssh -T git@github.com
```
#### Step 7: Debug SSH Connection (Optional)
```bash
ssh -vT git@github.com
```

### Clone the CRM Project Repository
```bash
git clone git@github.com:technohaven-company-limited/technohaven_automation.git
```

## PostgreSQL Setup

### Install PostgreSQL Latest Version
```bash
sudo apt -y install postgresql
```

### Start PostgreSQL Service
```bash
sudo systemctl status postgresql
sudo systemctl start postgresql
```

### Allow/Disallow Ports on Debian
Commands for managing firewall rules:
```bash
sudo apt-get install ufw
sudo ufw enable
sudo ufw status
sudo ufw allow 5432/tcp
sudo ufw delete allow <port>/tcp
sudo ufw reload
```

### Create a User for the CRM Project
```bash
sudo su postgres
psql
create role crm with login createdb createrole superuser password 'tc123';
```

## Virtual Environment Setup

### Install Python Virtual Environment
```bash
sudo apt install python3.11-venv
```
### Set Up the Virtual Environment
Change directory to your project root directory and run:
```bash
python3.11 -m venv venv
source venv/bin/activate
pip install wheel
python3.11 -m pip install -r requirements.txt
```

### Troubleshooting Dependencies
#### psycopg2 Errors
```bash
sudo apt-get update
sudo apt-get install -y build-essential libpq-dev python3-dev
pip install psycopg2-binary
pip install psycopg2
```

#### python-ldap Errors
```bash
sudo apt-get update
sudo apt-get install -y build-essential python3-dev libldap2-dev libsasl2-dev libssl-dev
pip install --upgrade pip setuptools wheel
pip install python-ldap --verbose
pip install ldap3
```

### Verify Installed Dependencies
```bash
pip list
```

## Configuration File

### Create a Configuration File
Path: `/etc/crm_tcl.conf`
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
addons_path=/opt/odoo-project/crm/odoo/addons, /opt/odoo-project/crm/odoo/custom
data_dir=/opt/odoo-project/crm/odoo/data_dir
logfile=/var/log/odoo/crm_tcl.log
log_level=debug
proxy_mode = True
http_enable = True
https_enable = True
http_port = 8071
https_port = 443

# HTTPS Configuration
secure_pkey_file =/etc/ssl/private/odoo71.key
secure_cert_file = /etc/ssl/certs/odoo71.crt
secure_ciphers = TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384
secure_ssl_redirect = True

[resource_settings]
limit_memory_hard = 1677721600
limit_memory_soft = 629145600
limit_request = 8192
limit_time_cpu = 600
limit_time_real = 1200
max_cron_threads = 1
workers = 60
```

## Service File

### Create a Service File
Path: `/etc/systemd/system/crm_tcl.service`
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
```

### Enable and Reload the Service
```bash
sudo systemctl enable crm_tcl.service
sudo systemctl daemon-reload
sudo systemctl start crm_tcl.service
```

## Database Backup and Restore

### Create a Database Dump File
```bash
pg_dump -U crm -h localhost -p 5432 crm_db > dump_crm_db_24Dec.sql
```

### Create a New Database
```bash
createdb -O owner_name new_db_name
```

### Restore the Database
```bash
psql new_db_name < dump.sql
```

### Access the CRM Instance
Visit: `http://192.168.10.26:8071`
