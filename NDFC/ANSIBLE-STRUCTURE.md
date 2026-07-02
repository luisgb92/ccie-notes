
# NDFC with Ansible Introduction

Following is the typical project directory structure of an Ansible project:

```yaml
ansible-project/
├── ansible.cfg       # Ansible configuration file
├── inventory         # Directory for inventory files
│   ├── hosts.yml     # Default inventory file
│   └── group_vars    # Directory to define variables for groups
│       └── group1.yml
│   └── host_vars     # Directory to define variables for specific hosts
│       └── host1.yml
├── playbooks         # Directory for playbook files
│   ├── playbook1.yml
│   └── playbook2.yml
├── roles             # Directory for Ansible roles
│   ├── role1         # A specific role
│   │   ├── defaults
│   │   ├── files
│   │   ├── handlers
│   │   ├── meta
│   │   ├── tasks
│   │   ├── templates
│   │   └── vars
│   └── role2         # Another specific role
├── files             # Directory for static files to copy to hosts
└── templates         # Directory for Jinja2 templates
```
## Ansible Config file

The ansible.cfg file is Ansible's configuration file. It controls **how Ansible behaves**, while the inventory tells Ansible **what devices to manage**.

Think of it this way:

|File | Purpose |
| --- | --- |
|ansible.cfg | Configures Ansible itself (behavior, defaults, plugins, paths, etc.) |
|inventory.yml |Lists the hosts and connection variables |
|playbook.yml | Defines the tasks to execute |

## Inventory

Ansible automates tasks on managed nodes or “hosts” in your infrastructure by using a list or group of lists known as inventory. Ansible composes its inventory from one or more ‘inventory sources’. 

Your inventory defines the managed nodes you automate and the variables associated with those hosts. You can also specify groups. 

Groups allow you to reference multiple associated hosts to target for your automation or to define variables in bulk. Once you define your inventory, you use patterns to select the hosts or groups you want Ansible to run against.

This YAML file is an Ansible inventory. An inventory tells Ansible which devices to manage and what connection variables to use for those devices.

Let's break it down.

```yaml

all:
  vars:
    ansible_connection: httpapi
    ansible_network_os: cisco.dcnm.dcnm
    ansible_httpapi_use_ssl: true
    ansible_httpapi_validate_certs: false
    ansible_httpapi_login_domain: local
    ansible_user: admin
    ansible_password: ins3965!

  children:
    dcnm_controllers:
      hosts:
        ndfc1:
          ansible_host: 10.31.125.222

        #ndfc2:
          #ansible_host: 192.168.2.11

```
### Top Level: all

all is the root group in every Ansible inventory.

Every host belongs to this group, so variables defined under all.vars are inherited by every host unless overridden.

```yaml
all:
```

### Global Variables (vars)

These variables apply to every host in the inventory.

```yaml
vars:
```

### Children Groups

Ansible organizes hosts into groups.

```yaml
children:
```

Here is one group:

```yaml
dcnm_controllers
```

### Group: dcnm_controllers

This group contains every DCNM/NDFC controller you want Ansible to manage.

```yaml
dcnm_controllers:
```

