---
layout: post
title:  "Setup Zero Gravity node with Ansible"
date:   2024-06-02 08:32:17 +0700
categories: crypto
---

![ansible_0g](/assets/img/ansible_0g.jpg)

Many services portals exist, most of them provide instructions about validating
nodes installation. However, when you think about redundancy and scaling,
the process must be automated, otherwise you will spend your life in running
similar shell commands again and again.

Ansilbe is a great tool that automates provisioning, configuration management,
application deployment and orchestration of you IT infrastructure.

I created a pretty simple playbook which installs, syncs and runs zero gravity
validating node. The code itself and instruction is in my git
[repo](https://github.com/masim05/web3-ansible-playbooks). Feel free to add
more playbooks and send PRs, you are welcome!