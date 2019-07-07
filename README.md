# static-routes

## Purpose

Compute nodes sometimes need multiple NICs. Frequent management of routing tables over multiple nodes in a cluster might be a cumbersome process if not automated.
This simple Ansible role will help this task, and will keep the single source ot truth (for routing tables) in inventory.
Every time the routing need to be extended, a set of meta-attributes can be configured and appended to the static routing table. These attributes are "name", "contact","date" and "comment", followed by tbe routing entries.

This role does not have intention to generic network management.

## Prerequisites

1  The full definition of interface is in the file `/etc/sysconfig/network-scripts/ifcfg-<interface-name>`

``` bash
[root@ip-172-201-1-10 network-scripts]# ll ifcfg-*
-rw-r--r--. 1 root root 193 Jul  6 19:03 ifcfg-eth0
-rw-r--r--. 1 root root 199 Jul  6 18:41 ifcfg-eth1
-rw-r--r--. 1 root root 254 May  3  2017 ifcfg-lo
```

2  The `/etc/sysconfig/network-scripts/ifcfg-<interface-name>` file contains entry: `GATEWAY=<ip4-address>`

``` bash
BOOTPROTO=dhcp
DEVICE=eth0
HWADDR=02:47:bc:0d:1d:9a
ONBOOT=yes
TYPE=Ethernet
USERCTL=no
DEFROUTE=yes
GATEWAY=172.201.1.1
```

Please note that GATEWAY parameter might not be configured OOTB, or by default.

3  Ansible inventory contains definition for routing configuration `inventories/<inventory>/all/static-routes-<interface-name>`

``` yaml
network_configuration_path: "/etc/sysconfig/network-scripts"
static_routes_eth0:
  - name: External routing example
    contact: contact@external.mail
    date: 2019.07.03
    comment: Example for eth0
    routes:
      - 172.15.1.2
      - 10.1.1.1
      - 10.2.2.2
```

## Usage

The Ansible playbook could be run as: `$ ansible-playbook static-routes.yml --ask-become-pass`

This will create new routes int the `/etc/sysconfig/network-scripts/route-<interface-eth0` file as defined in the inventory:

``` bash
# Project: External routing
# Contact: contact@external.mail
# Change: 2019.07.03
# Comment: Example for eth0 nic
172.15.1.2 via 172.201.1.1
10.1.1.1 via 172.201.1.1
10.2.2.2 via 172.201.1.1
```

And it will also restart network and set the routing for the node to:

``` bash
[root@ip-172-201-1-10 ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         172.201.1.1     0.0.0.0         UG        0 0          0 eth0
0.0.0.0         172.201.2.1     0.0.0.0         UG        0 0          0 eth1
10.1.1.1        172.201.1.1     255.255.255.255 UGH       0 0          0 eth0
10.2.2.2        172.201.1.1     255.255.255.255 UGH       0 0          0 eth0
172.15.1.2      172.201.1.1     255.255.255.255 UGH       0 0          0 eth0
172.201.1.0     0.0.0.0         255.255.255.0   U         0 0          0 eth0
172.201.2.0     0.0.0.0         255.255.255.0   U         0 0          0 eth1
[root@ip-172-201-1-10 network-scripts]#
```
