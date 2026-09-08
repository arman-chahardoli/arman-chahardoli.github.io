---
title: "Install Docker on Ubuntu with Ansible"
date: 2026-09-08 10:30:00 +0330
categories: [Docker, Install]
tags:
  - ansible
  - docker
  - install
image:
  path: assets/posts/images/install_docker_on_ubuntu_with_ansible.jpg
  alt: "Install Docker on Ubuntu with Ansible"
toc: true
comments: false
math: false
mermaid: true
pin: false
published: true
---
# Install Docker on Ubuntu with Ansible

An Ansible role I use to install Docker consistently across my Ubuntu machines.

It handles repository configuration, package installation, and Docker service management.

## Supported

- Ubuntu 20.04
- Ubuntu 22.04
- Ubuntu 24.04

## Installed

- Docker Engine
- Docker CLI
- containerd
- Docker Buildx
- Docker Compose plugin

## Role Structure

```text
install_docker_on_ubuntu/
├── tasks/
│   └── main.yml
└── templates/
    └── docker.sources.j2
```

## Example usage

Use the role in your playbook:

```yaml
---
- name: Install Docker
  hosts: docker_servers
  become: true

  roles:
    - install_docker_on_ubuntu
```

Then run your playbook

```bash
ansible-playbook -i inventory playbook.yml
```

## Verify

Check the installed Docker components:

```bash
docker --version
docker compose version
systemctl status docker
```

The role configures the official Docker APT repository dynamically based on the Ubuntu release and system architecture.

**Source:** [ansible-lab](https://github.com/arman-chahardoli/ansible-lab/tree/main/roles/install_docker_on_ubuntu)
