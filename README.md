# Ansible Environment Repository

This repository provides a structured Ansible environment where each individual site is managed as a separate `site.yaml` playbook. The setup ensures modularity, reusability, and easy management of configurations across different environments.

## Repository Structure

```
/project-root/
│── ansible/
│   ├── inventory/          # Ansible inventory files
│   │   ├── production      # Production inventory
│   │   ├── staging         # Staging inventory
│   │   ├── group_vars/     # Group-specific variables
│   │   ├── host_vars/      # Host-specific variables
│   ├── roles/              # Ansible roles
│   │   ├── common/         # Common configurations
│   │   ├── webserver/      # Webserver role
│   │   ├── database/       # Database role
│   │   ├── site1/          # Site1 roles
│   │   ├── site2/          # Site2 roles
│   │   ├── site3/          # Site3 roles
│   ├── site-configs/       # Per-site Ansible configurations
│   │   ├── site1.yaml      # Site 1 playbook
│   │   ├── site2.yaml      # Site 2 playbook
│   │   ├── site3.yaml      # Site 3 playbook
│   ├── playbooks/          # General playbooks
│   │   ├── deploy.yml      # Main deployment playbook
│   │   ├── rollback.yml    # Rollback playbook
│   ├── ansible.cfg         # Ansible configuration file
│   ├── requirements.yml    # Dependencies for roles
│   ├── README.md           # Documentation
│
│── semaphore/
│   ├── semaphore.yml       # Semaphore CI/CD pipeline configuration
│   ├── playbook-runner.yml # Semaphore playbook runner

```

## Inventory Configuration

### Production Inventory

#### `ansible/inventory/production`

---

## Setting Up Inventory Files

Ansible inventory files define which hosts belong to which environment (e.g., `production`, `staging`). These are located in the `inventory/` directory.

### Example Inventory File (`inventory/production`)
```ini
[webserver]
web1.example.com
web2.example.com

[database]
db1.example.com

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

## Site Variables

Each site can have its own configuration stored under `group_vars`.

#### `ansible/inventory/production/group_vars/site1.yml`

```yaml
site_name: "Site 1"
app_version: "1.0"
```

#### `ansible/inventory/production/group_vars/site2.yml`

```yaml
site_name: "Site 2"
app_version: "2.0"
```

## Playbooks

Each site can have a dedicated playbook for deployment and configuration.

#### `ansible/playbooks/site1.yaml`

```yaml
- name: Configure Site 1
  hosts: site1
  become: true
  roles:
    - common
    - site1
```

#### `ansible/playbooks/site2.yaml`

```yaml
- name: Configure Site 2
  hosts: site2
  become: true
  roles:
    - common
    - site2
```

#### `ansible/playbooks/common.yaml`

```yaml
- name: Apply common configurations
  hosts: all
  become: true
  roles:
    - common
```

## Roles

Each site can have its own role for specific configurations.

#### Example `ansible/roles/site1/tasks/main.yml`

```yaml
- name: Install required packages for Site 1
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Deploy Site 1 Application
  ansible.builtin.template:
    src: site1.conf.j2
    dest: /etc/nginx/sites-available/site1
```

#### Example `ansible/roles/site2/tasks/main.yml`

```yaml
- name: Install required packages for Site 2
  ansible.builtin.apt:
    name: apache2
    state: present

- name: Deploy Site 2 Application
  ansible.builtin.template:
    src: site2.conf.j2
    dest: /etc/apache2/sites-available/site2
```

## Setting Up Site-Specific Playbooks (site.yml)

Each siteX.yml file inside site-configs/ is a site-specific Ansible playbook that defines how a particular site should be deployed.

### Example siteX.yml
```
---
- name: Deploy Site X
  hosts: webserver
  become: true
  vars_files:
    - "../inventory/group_vars/webserver.yml"
    - "../inventory/host_vars/siteX.yml"

  roles:
    - role: common
    - role: webserver
    - role: database

  tasks:
    - name: Ensure webserver is installed
      apt:
        name: nginx
        state: present

    - name: Copy site configuration
      template:
        src: "../templates/siteX.conf.j2"
        dest: "/etc/nginx/sites-available/siteX.conf"

    - name: Enable site configuration
      file:
        src: "/etc/nginx/sites-available/siteX.conf"
        dest: "/etc/nginx/sites-enabled/siteX.conf"
        state: link

    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

## Setting Up Ansible Roles

Roles are reusable components that encapsulate tasks, handlers, and variables. They are stored in the `roles/` directory.

### Creating a New Role

Use the following command to create a new role:
```sh
ansible-galaxy init roles/new_role
```

### Example `roles/webserver/tasks/main.yml`
```yaml
---
- name: Install Nginx
  apt:
    name: nginx
    state: present
```

### Using a Role in a Playbook
```yaml
---
- name: Configure Webserver
  hosts: webserver
  become: true
  roles:
    - webserver
```

---

## Running the Deployment

To deploy a specific site:
```sh
ansible-playbook -i ansible/inventory/production ansible/site-configs/site1.yml
```

To deploy all sites together:
```sh
ansible-playbook -i ansible/inventory/production ansible/playbooks/deploy.yml
```

To rollback a deployment:
```sh
ansible-playbook -i ansible/inventory/production ansible/playbook/rollback.yml
```

## Ansible Configuration

Ensure Ansible is correctly configured to use the correct inventory.

#### `ansible/ansible.cfg`

```ini
[defaults]
inventory = ansible/inventory/production
host_key_checking = False
retry_files_enabled = False
roles_path = ansible/roles
```

## Running Playbooks

To deploy `site1` in the `production` environment:

```bash
ansible-playbook -i ansible/inventory/production/hosts ansible/playbooks/site1.yaml
```

To deploy `site2` in the `staging` environment:

```bash
ansible-playbook -i ansible/inventory/staging/hosts ansible/playbooks/site2.yaml
```

## Conclusion

This setup provides a scalable and modular way to manage multiple site deployments with Ansible. Each site has its own configuration, playbook, and role, ensuring a clean separation of concerns. Modify the repository structure as needed to fit your infrastructure requirements.
