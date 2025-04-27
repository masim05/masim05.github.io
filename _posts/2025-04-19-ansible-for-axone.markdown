---
layout: post
title:  "Setup Axone node with Ansible"
date:   2025-04-19 15:14:27 +0700
categories: crypto
---

![ansible_axone](/assets/img/ansible_axone.png)

### Motivation

When operating any production information system, one should think about high
availability and DDoS protection. In context of PoS validators, both can't be
achieved without having several nodes. More than that, sometimes you just need
to move your nodes to new VPS/VDS/DS. Having said that, one can state that it
is worth investing in automation of blockchain nodes' bootstrap. Ansible is a
great tool for it, and in this article I will show how to install, setup and
run a node in Axone testnet.

### Start

I have a [repo](https://github.com/masim05/web3-ansible-playbooks) containing
several ansible playbooks relevant for web3. It has a pretty straightforward
README, let's follow it together.

### Install ansible

In order to install ansible one would better to follow their official
[documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

### Setup controlled host

Next step is to prepare the controlled host, create `ansible` user with sudo permissions:

```bash
sudo adduser ansible
sudo usermod -aG sudo ansible
sudo su - ansible
mkdir -p ~/.ssh
touch ~/.ssh/authorized_keys
# Put your ssh public key into `~/.ssh/authorized_keys`
chmod 400 ~/.ssh/authorized_keys

# Create ansible working directory
sudo mkdir /var/ansible
sudo chown ansible:ansible /var/ansible
```

### Get playbooks

Then we need to clone the repo and install required dependencies:

```bash
git clone https://github.com/masim05/web3-ansible-playbooks.git
cd web3-ansible-playbooks
ansible-galaxy install -r requirements.yml
```

### Configure inventory

We need a valid `inventory.yml` file, one can check an example in the repo,
put proper `hosts` and `moniker`:

```yaml
axone:
    hosts:
        <address of the controlled host>:
    vars:
        user: axone
        port_prefix: 27
        go_version: 1.24.2
        git_tag: v10.0.0
        genesis_file_url: https://raw.githubusercontent.com/axone-protocol/networks/911b2d34631ac242e9ef3be577163653ed644726/chains/dentrite-1/genesis.json
        chain_id: axone-dentrite-1
        moniker: <your moniker>
        sync_rpc: https://axone-t-rpc.noders.services:443
        seeds: ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:17656
        systemd_service: axoned.service
```

You can also play with other parameters, eg, find working `sync_rpc` and `seeds`. Or
change `port_prefix` in case correspondent ports are being listened.

### Run it!

As simple as that:

```bash
ansible-playbook -i inventory.yml -v --ask-become-pass -l axone playbooks/axone/node.yml
```

That's it! It took about 8 minute for me to get things done:

```bash
=============================================================================== 
Build Axone ----------------------------------------------------------- 219.14s
Sync with network ------------------------------------------------------ 82.21s
Setup Cosmovisor ------------------------------------------------------- 73.12s
Configure general settings --------------------------------------------- 21.24s
Install go ------------------------------------------------------------- 10.03s
Setup axoned service ---------------------------------------------------- 9.98s
Install jq lz4 build-essential curl wget git ---------------------------- 9.25s
Clone Axone repo -------------------------------------------------------- 7.73s
Gathering Facts --------------------------------------------------------- 7.45s
Init Axone node --------------------------------------------------------- 7.11s
Reload systemd daemon --------------------------------------------------- 6.21s
Start axoned service ---------------------------------------------------- 5.94s
Enable axoned service --------------------------------------------------- 5.91s
Create system user ------------------------------------------------------ 5.65s
Show params ------------------------------------------------------------- 0.01s
Playbook run took 0 days, 0 hours, 7 minutes, 50 seconds
```

As a result we have a synced node (sync might take time), and it is up to you
to turn it into a sentry node or put validator's key there.
