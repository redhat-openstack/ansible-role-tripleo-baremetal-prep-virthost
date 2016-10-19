Role Name
=========

An Ansible role to set up a machine to host a virtual undercloud for a TripleO deployment on baremetal nodes.

Background
----------
TripleO quickstart creates two bridges named brovc and brext for external and overcloud network by default. The default network configuration resides in [tripleo-quickstart/roles/common/defaults/main.yml](https://github.com/openstack/tripleo-quickstart/blob/master/roles/common/defaults/main.yml#L85). The environment setup roles of quickstart follow template under [tripleo-quickstart/roles/environment/setup/templates/network.xml.j2](https://github.com/openstack/tripleo-quickstart/blob/master/roles/environment/setup/templates/network.xml.j2) to iterate over the variables and create libvirt network configuration xml. The external network is for management of undercloud itself from virthost, therefore nat is enabled to masquerade internal IP subnet (192.168.23.0/24 by default). The overcloud network is for overcloud nodes to communicate with undercloud, also known as provisioning network. In case of baremetal VM overcloud deployment, the overcloud nodes are deployed on same machine as undercloud and attached to overcloud network through the libvirt VM xml template under [tripleo-quickstart/roles/libvirt/setup/overcloud/templates/baremetalvm.xml.j2](https://github.com/openstack/tripleo-quickstart/blob/master/roles/libvirt/setup/overcloud/templates/baremetalvm.xml.j2#L29). However in case of baremetal overcloud deployment the overcloud network has to be extended to physical provisioning network by attaching the nic on provisioning network to bridge brovc. 

Requirements
------------

This role assumes that the host machine already has a nic on the provisioning network. The role plugs the nic into the bridge for overcloud network (brovc by default). 

Role Variables
--------------

**Note:** Make sure to include all environment file and options from your [initial Overcloud creation](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/7/html/Director_Installation_and_Usage/sect-Scaling_the_Overcloud.html)

- virthost_provisioning_interface: <eth1> --  NIC for the provisioning interface on the undercloud host
- virthost_provisioning_ip: <192.168.122.1> -- IP address for the provisioning interface on the undercloud host
- virthost_provisioning_netmask: <255.255.255.192> -- Netmask for the provisioning interface on the undercloud host
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
