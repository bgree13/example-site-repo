# example-site-repo

# Ansible Environment Repository

This repository provides a structured Ansible environment where each individual site is managed as a separate `site.yaml` playbook. The setup ensures modularity, reusability, and easy management of configurations across different environments.

## Repository Structure

```
ansible-environment-repo/
│── inventories/
│   ├── production/
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── site1.yml
│   │   │   ├── site2.yml
│   │   ├── hosts
│   ├── staging/
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── site1.yml
│   │   │   ├── site2.yml
│   │   ├── hosts
│── playbooks/
│   ├── site1.yaml
│   ├── site2.yaml
│   ├── common.yaml
│── roles/
│   ├── common/
│   ├── site1/
│   ├── site2/
│── ansible.cfg
│── requirements.yml
│── README.md
```

## Inventory Configuration

### Production Inventory

#### `inventories/production/hosts`

```ini
[site1]
site1-server1 ansible_host=192.168.1.10
site1-server2 ansible_host=192.168.1.11

[site2]
site2-server1 ansible_host=192.168.2.10
site2-server2 ansible_host=192.168.2.11
```

### Staging Inventory

#### `inventories/staging/hosts`

```ini
[site1]
staging-site1-server1 ansible_host=192.168.1.20

[site2]
staging-site2-server1 ansible_host=192.168.2.20
```

## Site Variables

Each site has its own configuration stored under `group_vars`.

#### `inventories/production/group_vars/site1.yml`

```yaml
site_name: "Site 1"
app_version: "1.0"
```

#### `inventories/production/group_vars/site2.yml`

```yaml
site_name: "Site 2"
app_version: "2.0"
```

## Playbooks

Each site has a dedicated playbook for deployment and configuration.

#### `playbooks/site1.yaml`

```yaml
- name: Configure Site 1
  hosts: site1
  become: true
  roles:
    - common
    - site1
```

#### `playbooks/site2.yaml`

```yaml
- name: Configure Site 2
  hosts: site2
  become: true
  roles:
    - common
    - site2
```

#### `playbooks/common.yaml`

```yaml
- name: Apply common configurations
  hosts: all
  become: true
  roles:
    - common
```

## Roles

Each site has its own role for specific configurations.

#### Example `roles/site1/tasks/main.yml`

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

#### Example `roles/site2/tasks/main.yml`

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

## Ansible Configuration

Ensure Ansible is correctly configured to use the correct inventory.

#### `ansible.cfg`

```ini
[defaults]
inventory = inventories/production
host_key_checking = False
retry_files_enabled = False
roles_path = roles
```

## Running Playbooks

To deploy `site1` in the `production` environment:

```bash
ansible-playbook -i inventories/production/hosts playbooks/site1.yaml
```

To deploy `site2` in the `staging` environment:

```bash
ansible-playbook -i inventories/staging/hosts playbooks/site2.yaml
```

## Version Control

To initialize and push the repository to Git:

```bash
git init
git add .
git commit -m "Initial Ansible environment setup"
git branch -M main
git remote add origin <your-repo-url>
git push -u origin main
```

## Conclusion

This setup provides a scalable and modular way to manage multiple site deployments with Ansible. Each site has its own configuration, playbook, and role, ensuring a clean separation of concerns. Modify the repository structure as needed to fit your infrastructure requirements.
