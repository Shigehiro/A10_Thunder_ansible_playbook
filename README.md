# A10 Ansible sample playbook

- [A10 Ansible sample playbook](#a10-ansible-sample-playbook)
  - [Descripton](#descripton)
  - [Tested Environment](#tested-environment)
  - [Walkthrough logs](#walkthrough-logs)

## Descripton

This is a sample ansible playbook to operate A10 Thunder.<br>
This playbook configures a health monitor, servers, service group and virtual server.

## Tested Environment

```
vThunder#show version | include ACOS
          64-bit Advanced Core OS (ACOS) version 6.0.5, build 51 (Aug-29-2024,16:50)
```

```
# podman exec a10-ansible ansible --version
ansible [core 2.14.14]
  config file = /root/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.9/site-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.9.18 (main, Sep  7 2023, 00:00:00) [GCC 11.4.1 20230605 (Red Hat 11.4.1-2)] (/usr/bin/python3)
  jinja version = 3.1.2
  libyaml = True
```

```
# podman exec a10-ansible ansible-galaxy collection list

# /root/.ansible/collections/ansible_collections
Collection     Version
-------------- -------
a10.acos_axapi 1.2.10
```

## Walkthrough logs

Edit inventory.ini and group_vars to meet your environment

Run the playbook.
```
ansible-playbook -i inventory.ini main.yml
```

On the thunder
```
vThunder#show running-config health monitor test_health_monitor
!Section configuration: 104 bytes
!
health monitor test_health_monitor
  retry 1
  method tcp port 80 send ping response contains ping
!
vThunder#
```

```
vThunder#show running-config slb service-group ansible-servicegroup
!Section configuration: 160 bytes
!
slb service-group ansible-servicegroup tcp
  method dst-ip-hash
  health-check test_health_monitor
  member server-1-tcp80 80
  member server-2-tcp80 80
!
vThunder#
```

```
vThunder#show running-config slb virtual-server VIP-TCP-80
!Section configuration: 121 bytes
!
slb virtual-server VIP-TCP-80 10.11.0.1
  port 80 tcp
    name ansible_demo
    service-group ansible-servicegroup
!
vThunder#
```