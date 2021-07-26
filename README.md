# Ansible - Scalable distribution and activation of ThousandEyes agents on Catalyst 9300
## IaaC approach to distributing Cisco Thousand Eyes Containers to Cisco 9300 IOS-XE platforms

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Set of scripts simplifies distribution, activation and configuration of Cisco ThousandEyes containers to supported Catalyst 9000 IOS-XE switches. 

## Features
- ThousandEyes package distribution (.tar)
- ThousandEyes Application Hosting container configuration
- Enterprise Proxy SSL Certificate distribution and activation in ThousandEyes container

## Requirements

- Ansible 
```sh
 ansible --version
ansible [core 2.11.2] 
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/auto/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/auto/.local/lib/python3.8/site-packages/ansible
  ansible collection location = /home/auto/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/auto/.local/bin/ansible
  python version = 3.8.10 (default, Jun  2 2021, 10:49:15) [GCC 9.4.0]
  jinja version = 2.10.1
  libyaml = True
```

- /etc/hosts file
```sh
cat /etc/hosts
10.1.1.5        c9300p27
10.1.1.15       c9300p28
```
