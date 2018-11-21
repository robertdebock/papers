# What makes an Ansible role?

It's not always easy to choose when to use an Ansible role or an Ansible playbook. This document describes how to choose beween a role and a playbook.

## Ansible role
An Ansible role is s set of Ansible tasks, typically written to add a function to a server. Some examples are:
- [java](https://galaxy.ansible.com/robertdebock/java)
- [nfs](https://galaxy.ansible.com/geerlingguy/nfs)
- [redis](https://galaxy.ansible.com/robertdebock/redis)

The structure of a role:
```text
.
├── tasks
│   └── main.yml # A list of tasks.
├── defaults
│   └── main.yml # Contains user settings.
├── handlers
│   └── main.yml # Used for tasks that are called using notify.
├── meta
│   └── main.yml # Galaxy uses metadata to information about display roles.
├── README.md    # Described to a user what this role is,
└── vars
    └── main.yml # Contains variables, not for user settings.
```

## Ansible playbook
An Ansible playbook is a list of pre_tasks, roles, tasks and post_tasks. In playbooks you target specific hosts.

A typical playbook may look like this:
```yaml
---
- name: build database servers
  hosts: databases
  gather_facts: yes
  become: yes

  vars:
    mysql_databases:
      - name: somedatabase
    mysql_users:
      - name: someuser
        password: somepassword
        priv: "somedatabase.*:ALL"

  pre_tasks:
    - name: notify team of start
      slack:
        token: "{{ slack_token }}"
        msg: "starting deployment of {{ inventory_hostname }}"

  roles:
    - robertdebock.bootstrap
    - robertdebock.mysql

  tasks:
    - name: notify team of end
      slack:
        token: "{{ slack_token }}"
        msg: "finished deployment of {{ inventory_hostname }}"
```

## Decide

Create an Ansible role when:
- You are [repeating yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) in playbooks.
- You want to publish your role to [GitHub](https://github.com/) or GitLab.
- Your playbook gets very long and does many tasks. (Move those tasks to a role)
- Dependencies and ordering is becoming relevant.
- You're describing a state that is idempotent.

Create an Ansible playbook when:
- You need to run tasks against a specific host.
- A task is unique and can't be applied anywhere else.
- Ordering (orchestration) is important.
- [Ansible facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variables-discovered-from-systems-facts) need to be found on one host and are reused on another host.
- You're describing an action, like "create report" or "restart service"
```
