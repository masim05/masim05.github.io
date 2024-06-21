---
layout: post
title:  "Expose API, RPC and gRPC endpoints over TLS with Ansible"
date:   2024-06-21 11:46:17 +0700
categories: crypto
---

In [one](/crypto/2024/05/25/public-endpoints.html) of my previous articles
I gave a manual guide about public endpoints setup. When you scale, such
a setup on every new server becomes a significant overhead. The best way
to tackle that is to automate it.

Like before, I use Ansible for infrastructure management. I prepared a playbook
which does the following:
 - generates certificates using Let's Encrypt
 - sets up Nginx to proxy node's endpoints

As simple as that! Just follow the
[guide](https://github.com/masim05/web3-ansible-playbooks/?tab=readme-ov-file#expose-public-endpoints)
and feel free to report issues and contribute!
