Openstack Service Builder Role
=========

This is a role that can be used to setup a multi-region/-project service
 on Openstack. It has been testet on 
 [SWITCHengines](https://www.switch.ch/de/engines/). 
 Your mileage on any other Openstack installation may vary. 

It can create and delete:
- networks 
- routers
- security groups
- servers
- data volumes 
- floating ips
- VIPs (for use with e.g. keepalived)
- server groups (e.g anti-affinity)

It allows for the definition of your virtual infrastructure in a declarative manner. E.g.

The definition for a VM with one data volume may look like this:

    os_server:
      my_server1: { ip: "10.0.1.10",  network: "my_network", flavor: "c1.small", image: "Ubuntu 16.04", root_size: "20", security_groups: "default,ssh",  hints: {}, floating_ip: '86.119.41.50' }
 
    os_volumes:
      my_server1:
        - { name: "data", name_prefix: "my_svc", state: "mounted", device: "/dev/vdb", size: "1024", fs: "xfs", owner: "root", group: "root", mode: "0755" }


You'd create it with:

    ansible-playbooks -i inventory playbooks/infra_set_up.yml


Example Playbook
----------------

Please have a look in the `<repo>/example_service/` directory for an example service.


License
-------

GPLv3

Author Information
------------------

Christian Schnidrig <christian.schnidrig@switch.ch>