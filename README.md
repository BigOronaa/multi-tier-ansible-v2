# Multi-Tier Application Deployment with Ansible - Documentation

## Project Overview
This project automates deployment and configuration of a multi-tier web application using Ansible. The application consists of:
- Nginx load balancer
- Python-based application servers
- MySQL database server

Environments: Development, Staging, Production.

CI/CD is handled via GitHub Actions with environment-specific variables and email notifications.

---

## Directory Structure
```
multi-tier-ansible-v2/
├── inventories/
│   ├── development/
│   │   ├── hosts.ini
│   │   └── group_vars/db.yml
│   ├── staging/
│   │   ├── hosts.ini
│   │   └── group_vars/db.yml
│   └── production/
│       ├── hosts.ini
│       └── group_vars/db.yml
├── roles/
│   ├── nginx/
│   ├── app/
│   └── db/
├── site.yml
└── .github/workflows/deploy.yml
```

---

## Step 1: Environment-Specific Configuration
**Development:**
- `inventories/development/hosts.ini`
- `group_vars/db.yml` with dev credentials

**Staging:**
- `inventories/staging/hosts.ini`
- `group_vars/db.yml` with staging credentials

**Production:**
- `inventories/production/hosts.ini`
- `group_vars/db.yml` with production credentials

Each environment contains unique database names, users, and passwords.

---

## Step 2: Roles and Playbooks
**Roles:**
- `nginx`: installs and ensures Nginx is running
- `app`: installs Python, pip, Flask, and runs the Flask app
- `db`: installs MySQL, ensures service is running, installs PyMySQL, creates database and users

**Playbook:**
- `site.yml` runs the roles in order: nginx, app, db

Example `db` tasks:
```yaml
- name: Install MySQL server
  apt:
    name: mysql-server
    state: present
    update_cache: yes
  become: yes

- name: Ensure MySQL service is running
  service:
    name: mysql
    state: started
    enabled: yes
  become: yes

- name: Ensure PyMySQL is available for Ansible MySQL modules
  pip:
    name: PyMySQL
    executable: /usr/bin/pip3
  become: yes

- name: Create database
  community.mysql.mysql_db:
    name: "{{ db_name }}"
    state: present
    login_user: root
    login_password: "{{ db_root_password }}"
  become: yes

- name: Create database user
  community.mysql.mysql_user:
    name: "{{ db_user }}"
    password: "{{ db_password }}"
    priv: "{{ db_name }}.*:ALL"
    state: present
    login_user: root
    login_password: "{{ db_root_password }}"
  become: yes
```

---

## Step 3: Local Testing
**Development Environment:**
```
ansible-playbook -i inventories/development/hosts.ini site.yml
```
- All tasks completed successfully
- `nginx`, `app`, `db` configured

**Staging Environment:**
```
ansible-playbook -i inventories/staging/hosts.ini site.yml
```
- Verified SSH connectivity
- All roles executed without errors

**Production Environment:**
```
ansible-playbook -i inventories/production/hosts.ini site.yml
```
- Deployment successful
- Verified database, app, and Nginx configuration

---

## Step 4: GitHub Actions Workflow
**File:** `.github/workflows/deploy.yml`
- Trigger: push to `master` (production) or `staging`
- Steps:
  1. Checkout code
  2. Setup Python and install Ansible
  3. Start SSH agent and add private key
  4. Add servers to `known_hosts`
  5. Set inventory based on branch
  6. Run `ansible-playbook`
  7. Send email notification

**Email Notification Step:**
- Uses `dawidd6/action-send-mail@v3`
- Sends workflow summary even if previous steps fail
- Configured with GitHub secrets for Gmail credentials

---

## Step 5: Error Handling
- Tasks have `retries` and `delay` configured for critical operations
- Ensures that temporary failures do not break the workflow

---

## Step 6: Deployment Verification
- SSH into servers or ping via Ansible
- Check Flask app is running on app servers
- Confirm MySQL database and users exist
- Verify Nginx is serving content
- Confirm email notification is received

---

## Notes
- World-writable directories trigger warnings in Ansible; ensure proper permissions
- Python interpreter warnings noted but not blocking
- All secrets managed via GitHub repository secrets



