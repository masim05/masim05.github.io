---
layout: post
title:  "Setup node exporter with Ansible"
date:   2024-06-16 21:46:17 +0700
categories: crypto
---

When operating production you do want to know about problems proactively, but
not from your customers. This is the point where monitoring jumps in.

Monitoring is usually devided into two parts: system metrics and business
metrics. System metrics stand for CPU usage, RAM consumption, IO statistics,
network and so on. Business metrics in context of validating nodes can refer
to amount missed blocks, proposed blocks, activity and so on.

Focus of this note is system monitoring. Good news, many open source
solutions are available these days. Node exporter, grafana and prometheus
is a very popular one.
[Node exporter](https://github.com/prometheus/node_exporter) exports the
metrics,
[prometheus](https://prometheus.io/) collects the data and
[grafana](https://grafana.com/) presents it. This
[artice](https://medium.com/@DanialEskandari/system-monitoring-with-prometheus-grafana-and-node-exporter-412027684564)
is a detailed guide how to do the setup.

When operating several production servers one should monitor each of them.
Thus, one should install node exporter on each of the servers. It may be
a bit annoying to install it again and again. So I decided to automate the
process with Ansible.

My repo contains several Ansilble playbooks simplifying validating nodes
maintenance. Feel free to use it in order to install node exporter
on every server you use, just follow the
[guide](https://github.com/masim05/web3-ansible-playbooks?tab=readme-ov-file#node-exporter-mac).
