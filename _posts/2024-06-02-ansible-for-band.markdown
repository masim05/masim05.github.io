---
layout: post
title:  "Setup Band node with Ansible"
date:   2024-12-12 11:25:38 +0700
categories: crypto
---

![ansible_band](/assets/img/ansible_band.png)

One more ansible playbook, this time it is for
[Band Protocol](https://www.bandprotocol.com/) node.

Everything is in the same
[repo](https://github.com/masim05/web3-ansible-playbooks), just clone it and
follow the guide for initial setup.
Then you need to create `inventory.yml` with some reasonable content:

```yaml
band:
    hosts:
        <your host here>:
    vars:
        user: band
        port_prefix: 27
        go_version: 1.24.2
        git_tag: v2.5.4
        genesis_file_url: https://raw.githubusercontent.com/bandprotocol/launch/master/laozi-mainnet/genesis.json
        bin_files_url: https://raw.githubusercontent.com/bandprotocol/launch/master/laozi-mainnet/files.tar.gz
        chain_id: laozi-mainnet
        moniker: <your moniker>
```

And run the command from `README.md`:

```bash
ansible-playbook -i inventory.yml -v --ask-become-pass -l band playbooks/band/node.yml
```

It worked for me within 50 min, the longest part was fetching the snapshot:

```bash
=============================================================================== 
Sync with network ---------------------------------------------------- 2602.00s
Build BandChain ------------------------------------------------------- 208.29s
Setup Cosmovisor ------------------------------------------------------- 79.12s
Install jq lz4 build-essential curl wget git --------------------------- 15.72s
Install go ------------------------------------------------------------- 10.22s
Clone BandChain repo ---------------------------------------------------- 9.88s
Setup bandd service ----------------------------------------------------- 9.67s
Gathering Facts --------------------------------------------------------- 8.99s
Init BandChain ---------------------------------------------------------- 7.00s
Reload systemd daemon --------------------------------------------------- 5.49s
Enable bandd service ---------------------------------------------------- 5.15s
Start bandd service ----------------------------------------------------- 4.62s
Create system user ------------------------------------------------------ 4.23s
Configure general settings ---------------------------------------------- 4.15s
Show params ------------------------------------------------------------- 0.01s
Playbook run took 0 days, 0 hours, 49 minutes, 34 seconds
```

The key of success is a snapshot from [highstakes](https://highstakes.ch/)
which is reliable like a Swiss watch.

After that you need to setup yoda, follow the official
[documentation](https://docs.bandchain.org/node-validators/run-node/joining-mainnet/installation#step-5-setup-yoda)
for it.

Hope it was useful and saved your time. Feel free to add more playbooks and
send PRs, you are welcome!
