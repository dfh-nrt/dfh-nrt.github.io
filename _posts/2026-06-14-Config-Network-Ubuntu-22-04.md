---
title: "Config Network Ubuntu 22.04"
date: 2026-06-14
tags:
  - Networking
  - Operating System
---

## Config Network Ubuntu 22.04

#### DHCP

```
path /etc/netplan/01-netcfg.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

#### Static

```
path /etc/netplan/01-netcfg.yaml

network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: false
      addresses: [192.168.1.10/24]
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
          on-link: true
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

Jangan lupa untuk apply konfigurasi diatas dengan ```netplan apply```
