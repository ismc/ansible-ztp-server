# Role Name

This role will setup a Redhat or Debian-based server to be a Zero-Touch Provisioning Server for network devices

### Requirements


Currently, this role requires RHEL/CentOS.  It will install the EPEL repository and all required dependancies

### Role Variables


| Name              | Default Value       | Description          |
|-------------------|---------------------|----------------------|
| `ztp_networks` | `no` | A list of dictionaries containing the netowrk from which clients will boot |
| `ztp_clients` | `no`  | A list of dictionaries describing the clients |
| `ztp_profiles` | `no` | A list of profiles that can be used by the clients |

Dependencies
------------

None

### Example Playbook


Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: ztp_servers
  become: yes
  vars:
    ztp_networks:
      - { cidr: 192.168.12.0/24, router: 192.168.12.1, dns: [8.8.8.8, 8.8.4.4], tftp_server: 192.168.12.10 }
    ztp_clients:
      - { hostname: isr, domain: foo.io, cidr: 192.168.12.30/24, mac: "ac:f2:c5:19:37:20", config_template: isr-1900.j2 }
      - { hostname: c3850, domain: foo.io, cidr: 192.168.200.31/24, mac: "70:6e:6d:4f:b0:47", config_template: c3850.j2, ztp_profile: 3850 }
    ztp_profile:
      3850:
        type: cisco
        option: 00:00:00:09:12:05:10:61:75:74:6f:69:6e:73:74:61:6c:6c:5f:33:38:35:30
  roles:
      - { role: ztp-server }

```

`ztp_networks` is a list of networks that are configured in the DHCP server.  If the networks are remote (i.e. not directly connected to the ZTP server), you need to configure the appropriate DHCP forwarding on the routers.

`ztp_clients` is a list of the clients that will be booting.  It specifies the name, domain, IP address to hand out, and the mac address from which the DHCP request will originate.  The `config-template` specifies the template that will be used to generate the configuration file for the device to pull.

`ztp_profile` can be used to specify commonly used attributes across clients

### Device ZTP Notes

#### Cisco Catalyst 3850

The Cisco Catalyst 3850 supports ZTP as documented [here](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwiNuZ2A96TWAhVKl1QKHcdNAfwQFggmMAA&url=https%3A%2F%2Fwww.cisco.com%2Fc%2Fen%2Fus%2Ftd%2Fdocs%2Fios-xml%2Fios%2Fdatamodels%2Fconfiguration%2Fxe-16%2Fdata-models-xe-16-book%2Fzero-touch-provision.pdf&usg=AFQjCNFczoNUlk9kNrLzDy7_wIK6Re8TBQ).  The following is an ISC DHCP block for a Cat 3850 that will support handing out IP address, name, and configuration file.  It will also upgrade the switch software:

```
host c3850.foo.io {
  hardware ethernet 70:6e:6d:4f:b0:47;
  fixed-address 192.168.200.31;
  option host-name "c3850.foo.io";
  option bootfile-name "configs/c3850.foo.io";
  send option-125 00:00:00:09:12:05:10:61:75:74:6f:69:6e:73:74:61:6c:6c:5f:33:38:35:30;
  filename "configs/c3850.foo.io";
 }
 ```

 * `option host-name` specifies the configuration filename.
 * `send option-125` specifies a file that contains the name of the image that the device should use for upgrading.

This value is interpreted as:

* 00-00-00-09 — Enterprise Number (Cisco Value)
* 12 — Option 125 Data Len
* 05 — Sub Option Code
* 10 — Sub Option Length
* 61:75:74:6f:69:6e:73:74:61:6c:6c:5f:33:38:35:30 — Sub Option Data ( txt – ASCII to HEX conversion)

In this case, the option tells the switch to access file `autoinstall_3850` for the name of the software to use for the upgrade.  In this case that file contains:

```
sw/cat3k_caa-universalk9.SPA.03.06.06.E.152-2.E6.bin
```

The switch then performs another TFTP operation to retrieve the software.

License
-------

GPL-3
