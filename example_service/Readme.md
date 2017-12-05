Sample Service for Openstack Service Builder Role
=========

This is a simple example of how one can build the virtual infrastructure for an imaginary service. 
(The service in this example does nothing useful).

It should be possible to run the example right from this directory provided you added some config file as described in "Setup"

Setup
=========

## Openstack Credentials
You need to add a file with your openstack credentials:
Copy the file `<repo>/group_vars/all/user.sample.yml` to `<repo>/group_vars/all/user.yml` and add your credential to it.

Edit the region information in the file `<repo>/group_vars/infra.site1.yml`

    os_region: "ZH"



## Bootable Image
Edit image name in `<repo>/group_vars/all/openstack.yml` to reflect name of a suitable image on your Openstack instance.

## Floating IP
While it is not strictly necessary to do this beforehand, it is easier to do it this way. 
Allocate a floating ip which you will be using as jumphost ip.

Edit the file `<repo>/ssh_config` and change the jumphost ip:

    Host jumphost.zhdk
      HostName 86.119.41.50
 
## Inventory

It is possible to have several inventories: e.g. `production` `staging` etc.

The inventory lists the server names of all servers. It does not matter if they can be resolved via DNS or not. Note that 
the inventory does not list any IPs. They are specified in the site configurations.

If you have multiple sites, take care to organize the servers similar to the example provided. The servers must 
belong to two groups, one per site and one per server type.
This is such, that it is possible to create appropriate variable files in group_vars for each type of server.

There needs to be an openstack group. The servers in there are pseudo servers which can all point to the 
same server or localhost on which you have the Openstack clients installed. As with regular servers, the pseudo servers
have to be in a group per site.

The group servers must contain all servers but not the pseudo openstack servers.


## Site Configuration

Site Configuration files e.g. `infra.site1.yml` defined all the ressources that should be created for the site.
`infra.site1.yml` is a simple site to get started. `infra.site2.yml` is a more complex site showcasing the possible features of this role.

The two site configs belong to `production`. Suppose you wanted to have a staging as well, then you'd create a similar `staging` inventory along with 
the corresponding `infra.staging_site1.yml` `infra.staging_site2.yml` configs.

Mandatory config data:
- os_network
- os_server

Optional config data:

- os_security_group
- os_server_group
- os_volumes
- os_vip


### IPv6

IPv6 is optional. If you choose to use it, then you need to add the network prefix you got in the 
file `<repo>/group_vars/infra.site1.yml` after having created the network and before you create any VMs.

    ipv6_prefix: "2001:620:5ca1:4002:"

Run Playbooks
=========

Now it is time to run the playbooks:

Build everything on site1:

    ansible-playbook -i production infra_set_up.yml --limit=*.site1
    ansible-playbook -i production service_playbook.yml --limit=*site1

It would be possible to add the openstack-service-builder role to the service_playbook as well. That would allow you to 
build a VM and deploy the service software in one go.

Delete everything on site1: 

    ansible-playbook -i production infra_clean_up.yml --limit=*.site1
    
Rebuild a single server on site1: 

    ansible-playbook -i production infra_clean_up.yml --limit=myserver1.site1
    ansible-playbook -i production infra_set_up.yml --limit=myserver1.site1
    ansible-playbook -i production service_playbook.yml --limit=myserver1.site1
    


Tags
=========

Often you may not want to run the whole playbooks. Therefore there are many useful tags available:


- infra: everything, useful if you add the role to the service playbooks.
- os_network
- os_router
- os_sec_group
- os_server_group
- os_vip
- os_volume: create/delete root volume
- os_port
- os_server
- os_vip_port: apply vip to a server
- os_floating_ip
- os_server_all: shortcut for os_volume,os_port,os_server,os_vip_port,os_floating_ip
- os_data: create/delete data volumes
- os_attach: attach/detach data volumes

