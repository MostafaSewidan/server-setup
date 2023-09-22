# Server Setup with Ansible

## What is Ansible?
Ansible is an open-source software provisioning, configuration management, and application-deployment tool enabling infrastructure as code. It runs on many Unix-like systems, and can configure both Unix-like systems as well as Microsoft Windows.

## How to install Ansible?
Ansible is an agentless automation tool that you install on a control node. From the control node, Ansible manages machines and other devices remotely (by default, over the SSH protocol).

[Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

[How to install Ansible on Windows?](https://geekflare.com/ansible-installation-windows/)

## What are Ansible roles?
Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users. Role directory structure. Storing and finding roles.

## Configuration steps
Roles
```
- nginx
- mysql
- php
- phpMyAdmin
- wordpress
```

## TODOs

#### h5bp role
### fail2ban
### ssl
### creating tocaan user