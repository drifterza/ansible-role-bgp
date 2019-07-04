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
| Debian | Debian  | Stretch    | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |
| Debian | Ubuntu  | Bionic    | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |
| Debian | Ubuntu  | Xenial    | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |
| Debian | Ubuntu  | Trusty    | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |
| RedHat | Centos  | 7         | [![x86_64](http://img.shields.io/badge/x86_64-passed-006400.svg?style=flat)](#) |

## Requirements

- ansible >= 2.x

## Molecule Testing

### Requirements
- python-virtualenv >= 15.x
- python-dev >= 2.7.16
- molecule == 2.19

```bash
mkdir Development
git clone git@gitlab.platform.is:platformengineering/ansible-roles/ansible-role-bgp.git Development/
cd Development/ansible-role-bgp
virtualenv molecule-testing
source molecule-testing/bin/activate
pip install -r test-requirements.txt
```
### Run Molecule

```bash
molecule create
molecule converge
```
This role currently uses centos7 as the molecule image. If you prefer you can change the `molecule/default/molecule.yaml` to test Ubuntu etc...

## Role Variables

| variable | default | description |
|--:|:-:|:--|
| bird_conf_dir | `/etc/bird` | default configuration directory |
| bird_router_id   | `127.0.0.1` | This is just a stub, please see the example to use the default IPv4 from ansible facts |
| bird_gather_net_facts | `true` | Force an update of the network facts before building the configuration. This is useful when protocol auto detection is used,because the network may have changed since facts were last gathered if they have been cached a while. |
| bird_ipv4_enabled | `detect` | Auto-enable bird/bird6 if an ipv4/ipv6 default route is present (Set to true/false to override auto detection) |
| bird_ipv6_enabled | `detect` | Auto-enable bird/bird6 if an ipv4/ipv6 default route is present (Set to true/false to override auto detection) | 


### Debian-only

| variable | default | description |
|--:|:-:|:--|
| bird_apt_repo_url | `http://bird.network.cz/debian/` | default repository url |

### Ubuntu-only

| variable | default | description |
|--:|:-:|:--|
| bird_apt_repo_url | `http://ppa.launchpad.net/cz.nic-labs/bird/ubuntu` | default repository url |

### Redhat-only

| variable | default | description |
|--:|:-:|:--|
| bird_package_source | `epel` | default repository to fetch packages |
| bird_packages | `bird` | This will install bird 1.6.6 as of this writing |
| bird_ipv6_enabled | `skip` | RHEL uses a combined systemd unit for bird/bird6 called "bird" which manages both daemons. |

## Playbooks

Example for CentOS using Bird 2.0.4

    - hosts: servers
      roles:
         - role: ansible-role-bgp
           bird_packages: bird2
           bird_package_source: bird
           bird_router_id: "{{ ansible_default_ipv4['address'] }}"
