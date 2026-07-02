
# NDFC with Ansible Introduction

Following is the typical project directory structure of an Ansible project:

```yaml
ansible-project/
├── ansible.cfg       # Ansible configuration file
├── inventory         # Directory for inventory files
│   ├── hosts         # Default inventory file
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

This YAML file is an Ansible inventory. An inventory tells Ansible which devices to manage and what connection variables to use for those devices.

Let's break it down.