# ansible-role-bgp

## Description

BIRD is an open-source implementation for routing Internet Protocol packets on Unix-like operating systems. It was developed as a school project at the Faculty of Mathematics and Physics, Charles University, Prague and is distributed under the GNU General Public License.

BIRD supports Internet Protocol version 4 and version 6 by running separate daemons. It establishes multiple routing tables and uses BGP, RIP, and OSPF routing protocols, as well as statically defined routes. Its design differs significantly from GNU Zebra and Quagga. Currently BIRD is included in many Linux distributions, such as Debian, Ubuntu and Fedora/RHEL.

BIRD is used in several Internet exchanges, such as the London Internet Exchange (LINX), LONAP, DE-CIX and MSK-IX as a route server, where it replaced Quagga because of its scalability issues. According to the 2012 Euro-IX survey, BIRD is the most used route server amongst European Internet exchanges.

In 2010, CZ.NIC, the current sponsor of BIRD development, received the LINX Conspicuous Contribution Award for contribution of BIRD to the advancement in route server technology

## Design

BIRD implements an internal routing table to which the supported protocols connect. Most of these protocols import network routes to this internal routing table and also export network routes from this internal routing table to the given protocol. This way information about network routes is exchanged among different routing protocols.

Using the kernel protocol this internal routing table may be connected to the actual kernel routing table. This allows BIRD to export network routes from its internal routing table to the kernel routing table and optionally also learn about network routes from the kernel routing table (created externally by the administrator or by other means) and import these routes into its internal routing table.

Filters may be used to control what network routes are imported into the internal routing table or exported to the given protocol. Network routes may be accepted, rejected or modified using filters.

BIRD also supports multiple internal routing tables and multiple instances of supported protocol types. Protocols may be connected to different internal routing tables, these internal routing tables may exchange information about network routes they contain (controlled by filters) and each of these internal routing tables may be connected to a different kernel routing table thus allowing for policy routing.

Configuration is done by editing the configuration file and telling BIRD to reconfigure itself. BIRD changes to the new configuration without the need to restart the daemon itself and restarts reconfigured protocols only if necessary. There is also an option to do a soft reconfiguration, which doesn't restart protocols but may leave some stale information such as changed filters not filtering out already exported network routes.

## Tests

| Family | Distribution | Version | Test Status |
|:-:|:-:|:-:|:-:|
| RedHat | Centos  | 7         | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |

## Requirements

- ansible >= 2.x

## Molecule Testing

### Requirements
- python-virtualenv >= 15.x
- python-dev >= 2.7.16
- molecule == 2.19
- ansible-logrotate

```bash
git clone git@gitlab.platform.is:platformengineering/ansible-roles/ansible-role-bgp.git ansible-role-bgp
cd ansible-role-bgp
virtualenv molecule-testing
source molecule-testing/bin/activate
pip install -r test-requirements.txt
ansible-galaxy install -r requirements.yml
```
### Run Molecule

```bash
molecule create
molecule converge
```

## Role Variables

| variable | default | description |
|--:|:-:|:--|
| bird_conf_dir | `/etc/bird` | default configuration directory |
| bird_password_enabled | false | no protocol passwords |
| bird_package_state | latest | Latest release |
| bird_router_id | `"{{ ansible_default_ipv4['address'] }}"` | defaults to first ip address |
| bird_gather_net_facts | true | Force an update of the network facts before building the configuration. This is useful when protocol auto detection is used, because the network may have changed since facts were last gathered if they have been cached a while.|
| bird_ipv4_enabled | detect | Auto-enable bird if an ipv4 default route is present
| bird_ipv6_enabled | detect | Auto-enable bird if an ipv6 default route is present |
| bird_bgp_rr_enabled | false | enable route reflector mode |
| bird_bgp_rs_enabled | false | enable route server mode |
| bird_bgp_static_enabled | false | enable static routes |
| bird_syslog_location | `"/var/log/bird.log"` | default location for the logs |
| bird_ipv4_cidr | `127.0.0.1/8` | defaul ipv4 cidr |
| bird_ipv6_cidr | `fdd1:ac8c:0557:7ce1::/64` | default ipv6 cidr |
| bird_neighbor_ipaddress | `"127.0.0.1"` | default neighbour address |
| bird_friendly_name | `"bird"` | default neighbour name |
| bird_password | `"secret"` | default password |

## Playbooks

Example for using Bird

    - hosts: serverA
      roles:
         - role: ansible-role-bgp
           enable_firewalld: true
           bird_bgp_rr_enabled: true
           bird_bgp_rs_enabled: false
           bird_bgp_static_enabled: false
           bird_password_enabled: true
         - role: '~/.ansible/roles/ansible-logrotate'
           logrotate_scripts:
             - name: bird-logrotate
                path: /var/log/bird.log
                options:
                  - daily
                  - weekly
                  - size 25M
                  - rotate 7
                  - missingok
                  - compress
                  - delaycompress
                  - copytruncate
      vars:
        rr_ipv4:
          172.17.0.0:
            bird_friendly_name: "bird_rr_ipv4" # Ensure the friendly name uses lowercase and underscores to create long names #
            bird_local_as_ipv4: 65000 # Used to override the template import
            bird_neighbor_ipaddress_ipv4: 172.17.0.3 # Only internal neighbor can be RR client #
            bird_neighbor_as_ipv4: 65000 # ASN needs to be the same ASN as the peer to act as a RR client #
        rr_ipv6:
          2001:db8:1::242:ac11:2:
            bird_friendly_name: "bird_rr_ipv6" # Ensure the friendly name uses lowercase and underscores to create long names #
            bird_local_as_ipv6: 65000 # Used to override the template import
            bird_neighbor_ipaddress_ipv6: "2001:db8:1::242:ac11:3" # Only internal neighbor can be RR client #
            bird_neighbor_as_ipv6: 65000 # ASN needs to be the same ASN as the peer to act as a RR client #

    - hosts: serverB
      roles:
         - role: ansible-role-bgp
           enable_firewalld: true
           bird_bgp_rr_enabled: true
           bird_bgp_rs_enabled: false
           bird_bgp_static_enabled: false
           bird_password_enabled: true
         - role: '~/.ansible/roles/ansible-logrotate'
           logrotate_scripts:
             - name: bird-logrotate
                path: /var/log/bird.log
                options:
                  - daily
                  - weekly
                  - size 25M
                  - rotate 7
                  - missingok
                  - compress
                  - delaycompress
                  - copytruncate
      vars:
        rr_ipv4:
          172.17.0.0:
            bird_friendly_name: "bird_rr_ipv4" # Ensure the friendly name uses lowercase and underscores to create long names #
            bird_local_as_ipv4: 65000 # Used to override the template import
            bird_neighbor_ipaddress_ipv4: 172.17.0.2 # Only internal neighbor can be RR client #
            bird_neighbor_as_ipv4: 65000 # ASN needs to be the same ASN as the peer to act as a RR client #
        rr_ipv6:
          2001:db8:1::242:ac11:2:
            bird_friendly_name: "bird_rr_ipv6" # Ensure the friendly name uses lowercase and underscores to create long names #
            bird_local_as_ipv6: 65000 # Used to override the template import
            bird_neighbor_ipaddress_ipv6: "2001:db8:1::242:ac11:2" # Only internal neighbor can be RR client #
            bird_neighbor_as_ipv6: 65000 # ASN needs to be the same ASN as the peer to act as a RR client #