Role Name
=========

This role will setup a server to be a Zero-Touch Provisioning Server for network devices

Requirements
------------

Currenlty, this role requires RHEL/CentOS.  It will install the EPEL repository and all required dependancies

Role Variables
--------------


| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `ztp_networks` | `no` | A list of dictionaries containing the netowrk from which clients will boot |
| `ztp_clients` | `no`  | A list of dictionaries describing the clients |

Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

- hosts: ztp_servers
  become: yes
  vars:
    ztp_networks:
      - { cidr: 192.168.12.0/24, router: 192.168.12.1, dns: [8.8.8.8, 8.8.4.4], tftp_server: 192.168.12.10 }
    ztp_clients:
      - { hostname: isr, domain: home.bus8.io, cidr: 192.168.12.30/24, mac: "ac:f2:c5:19:37:20", config_template: isr-1900.j2 }
  roles:
    - { role: ztp-server }

License
-------

MIT
