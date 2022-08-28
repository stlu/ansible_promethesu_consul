<p><img src="https://cdn.worldvectorlogo.com/logos/prometheus.svg" alt="prometheus logo" title="prometheus" align="right" height="60" /></p>

# Ansible Role: ansible_prometheus_consul

[![License](https://img.shields.io/badge/license-MIT%20License-brightgreen.svg)](https://opensource.org/licenses/MIT)

## Description

Deploy [Prometheus](https://github.com/prometheus/prometheus) [Consul](https://learn.hashicorp.com/consul) monitoring system using ansible.

## Requirements

- Ansible >= 2.10 (It might work on previous versions, but we cannot guarantee it)
- https://github.com/ansible-community/ansible-consul

## Role Variables

All variables which can be overridden are stored in [defaults/main.yml](defaults/main.yml) file as well as in table below.

| Name           | Default Value | Description                        |
| -------------- | ------------- | -----------------------------------|
| `prometheus_version` | 2.38.0 | Prometheus package version. Also accepts `latest` as parameter. |
| `prometheus_skip_install` | false | Prometheus installation tasks gets skipped when set to true. |
| `prometheus_binary_install_dir` | "/usr/local/bin" | Prometheus installation directory where `prometheus` binaries are stored on host on which ansible is ran. |
| `prometheus_config_dir` | /etc/prometheus | Path to directory with prometheus configuration |
| `prometheus_db_dir` | /var/lib/prometheus | Path to directory with prometheus database |
| `prometheus_config_file` | "prometheus.conf.j2" | Variable used to provide custom prometheus configuration file in form of ansible template |
| `prometheus_node_exporter_version` | 1.3.1 | Prometheus node_exporter package version. |
| `prometheus_node_exporter_binary_install_dir` | "/usr/local/bin" | Prometheus node_exporter installation directory |

## Installation and Usage tasks

## Inventory.ini example
```
# Prometheus server
[prometheus]
promserver1 ansible_host=172.17.0.3

[node_exporter]
exporter1 ansible_host=172.17.0.7 consul_node_role=client

# Consul Server
[consul_instances]
consulserver1 ansible_host=172.17.0.5 consul_node_role=bootstrap consul_bind_address=172.17.0.5

[consul_group_name] 
consulserver1 ansible_host=172.17.0.5 consul_node_role=bootstrap
exporter1 ansible_host=172.17.0.7 consul_node_role=client
```

## Create server prometheus (port 9090)

```yaml
---
# ansible-playbook  --tags=prometheus_server_configure
- hosts: prometheus
  become: true
  gather_facts: yes
  become: yes

  roles:
  - ansible_prometheus_consul

```
Verify via browser http://localhost:9090/targets



## Create server consul (port 8500 8600/udp)
docker run -d --privileged --name nodo2Deb11 -p 8500:8500 -p 8600:8600/udp node_debian11_ssh
```yaml
---
- hosts: consul_instances
  become: true
  gather_facts: yes
  become_user: root
  any_errors_fatal: true

  vars:
    consul_version: latest
    consul_iface: eth0
  roles:
  - ansible-consul

```
Verify via shell consul members
Verify via browser http://localhost:8500/



## Create a node with agent consul and prometheus node_exporter (port 9100)
docker run -d --privileged --name nodo3Deb11 -p 9100:9100 node_debian11_ssh 
```yaml
---
- hosts: node_exporter
  become: true
  gather_facts: yes
  become_user: root

  vars:
    consul_version: latest
    consul_iface: eth0
    consul_node_name: consul-node1
    consul_node_role: client

  roles:
  - ansible-consul
```
Verify that /opt/consul/serf/local.keyring is identical on every node and in /etc/consul/consul.json ("encrypt")
using: consul keyring -list
Verify via shell consul members


```yaml
---
# ansible-playbook   --tags=prometheus_node_exp_install
- hosts: node_exporter 
  become: true
  gather_facts: yes
  become: yes

  roles: 
  - ansible_prometheus_consul
```
Verify via web  localhost:9100/metrics

## Decomission the node 

```yaml
---
## ansible-playbook  --tags=prometheus_node_remove
- hosts: node_exporter
  become: true
  gather_facts: yes
  become: yes

  roles:
  - ansible_prometheus_consul
```
