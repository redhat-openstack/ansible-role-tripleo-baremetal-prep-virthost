Role Name
=========

An Ansible role to set up a machine to host a virtual undercloud for a TripleO deployment on baremetal nodes.

Background
----------
TripleO quickstart creates two bridges named brovc and brext for external and overcloud network by default. The default network configuration variables reside in [tripleo-quickstart/roles/common/defaults/main.yml](https://github.com/openstack/tripleo-quickstart/blob/master/roles/common/defaults/main.yml#L85). The environment setup roles of quickstart follow template under [tripleo-quickstart/roles/environment/setup/templates/network.xml.j2](https://github.com/openstack/tripleo-quickstart/blob/master/roles/environment/setup/templates/network.xml.j2) to iterate over the variables and create libvirt network configuration xml. The external network is for management of undercloud itself from virthost, therefore nat is enabled to masquerade internal IP subnet (192.168.23.0/24 by default). The overcloud network is for overcloud nodes to communicate with undercloud, also known as provisioning network. In case of baremetal VM overcloud deployment, the overcloud nodes are deployed on same machine as undercloud and attached to overcloud network through the libvirt VM xml template under [tripleo-quickstart/roles/libvirt/setup/overcloud/templates/baremetalvm.xml.j2](https://github.com/openstack/tripleo-quickstart/blob/master/roles/libvirt/setup/overcloud/templates/baremetalvm.xml.j2#L29). However, in case of baremetal overcloud deployment the overcloud network has to be extended to physical provisioning network by attaching the nic on provisioning network to bridge brovc.

After the NIC is attached to bridge by this role, it becomes slave of the bridge therefore it's IP address cannot be used to connect virthost to provisioning network. Instead the bridge itself should be given an IP address, to do that default network variables should be overridden by:

```
external_network_cidr: 192.168.23.0/24
overcloud_network_cidr: 10.0.0.0/24
networks:
  - name: external
    bridge: brext
    forward_mode: nat
    address: "{{ external_network_cidr|nthhost(1) }}"
    netmask: "{{ external_network_cidr|ipaddr('netmask') }}"
    dhcp_range:
      - "{{ external_network_cidr|nthhost(10) }}"
      - "{{ external_network_cidr|nthhost(50) }}"
    nat_port_range:
      - 1024
      - 65535

  - name: overcloud
    bridge:  brovc
    address: "{{ overcloud_network_cidr|nthhost(1) }}"
    netmask: "{{ overcloud_network_cidr|ipaddr('netmask') }}"
```
This permits the undercloud to communicate with virthost through the bridge's IP address. It also allows Ironic to communicate with virthost's hypervisor which can be used for hybrid overcloud deployment, where overcloud could be distributed between baremetal and virtual nodes at same time.   

Requirements
------------

This role assumes that the host machine already has a nic on the provisioning network. The role plugs the nic into the bridge for overcloud network (brovc by default). 

Role Variables
--------------

**Note:** Make sure to include all environment file and options from your [initial Overcloud creation](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/sect-Scaling_the_Overcloud.html)

- virthost_provisioning_interface: <eth1> --  NIC for the provisioning interface on the undercloud host
- virthost_provisioning_hwaddr: <52:54:00:00:76:00> -- MAC address the provisioning interface on the undercloud host
- working_dir: <'/home/stack'> -- working directory for the role.


Dependencies
------------

The playbook included in this role calls https://github.com/redhat-openstack/ansible-role-tripleo-validate-ipmi and https://github.com/redhat-openstack/ansible-role-tripleo-baremetal-overcloud.

Example Playbook
----------------

  1. Sample playbook to call the role

    - name: Prepare the host for PXE forwarding
      hosts: virthost
      gather_facts: no
      roles:
        - ansible-role-tripleo-baremetal-prep-virthost

License
-------

Apache

Author Information
------------------

RDO-CI Team
