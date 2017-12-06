Sample Service for Openstack Service Builder Role
=========

This is a simple example of how one can build the virtual infrastructure for an imaginary service. 
(The service in this example does nothing useful, it is merely a demo).

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

    os_image:
      xenial: "Ubuntu Xenial 16.04 (SWITCHengines)"


## Floating IP

### Manual Allocation
While it is not strictly necessary to do this beforehand, it is easier to do it this way. 
Allocate a floating ip which you will be using as jumphost ip.

    openstack floating ip create public
    
Extract the info: 

    | floating_ip_address | 86.119.41.50                         |

Edit the file `<repo>/ssh_config` and change the jumphost ip:

    Host jumphost.site1
      HostName 86.119.41.50

If you plan to use site2 too, do the same thing for site2 as well.

### Automatic Allocation
Alternatively you could leave the floating ip field empty. A new floating ip will then be allocated on VM creation and deallocated on VM deletion. E.g.

      myserver1.site2: { ip: "{{ipv4_prefix}}.10", network: "{{os_network_name}}", flavor: "c1.small", image: "{{os_image['xenial']}}", root_size: "20", security_groups: "{{site}}-ssh", hints: {group: "{{ os_server_group['server_group1'] }}"}, floating_ip: '' }

Note that you'll get a different floating ip each time if you want them to be allocated automatically.

You could also have an IP automatically allocated and then after first VM creation edit the site config and add the floating ip to the server definition.

 
## Inventory

It is possible to have several inventories: e.g. `production` `staging` etc.

The inventory lists the server names of all servers. It does not matter if they can be resolved via DNS or not. Note that 
the inventory does not list any IPs. They are specified in the site configurations `<repo>/group_vars/infra.*.yml`

If you have multiple sites, take care to organize the servers in the inventory similar to the example provided. The servers must 
belong to two groups each, one 'per site group' and one 'per server type group'. e.g. on site1, `myserver1.site1` 
belongs to `server_group1` (a server type group) and `infra.site1` (a site group).
Thanks to this, it is possible to create appropriate variable files in `group_vars` for each type of server and for each site.

There needs to be an openstack group for 'pseudo openstack' servers, one for each site. The servers in there are pseudo servers which can all point to the 
same physical server/VM or localhost on which you must have the Openstack clients installed. As with regular servers, the pseudo openstack servers
have to be in a group per site.

The group servers must contain all servers but not the pseudo openstack servers.


## Site Configuration

Site Configuration files e.g. `infra.site1.yml` define all the ressources that should be created for the site.
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

## Site1 (simple site)

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
    

## Site2 (complex site)

First setup everything that is not a VM: (limit playbook to pseudo openstack server)

    ansible-playbook -i production infra_set_up.yml --limit=os.site2


#### IPv6
Get the ipv6 subnet details:

    openstack subnet list
    openstack subnet show site2_ipv6

Take the network prefix from

     | cidr              | 2001:620:5ca1:2e4::/64                                     |

Add the network prefix to `<repo>/group_vars/infra.site2.yml`

    ipv6_prefix: "2001:620:5ca1:2e4:"

#### Server Groups

Get the uuid of the server group created:

    openstack server group list
    
Extract the ID from the output:

    | c489e94f-0c1a-40c1-a979-a897fcc6db88 | server_group1 | anti-affinity |

and add it to `<repo>/group_vars/infra.site2.yml`

    os_server_group:
      list:
        - { name: "server_group1", policies: ['anti-affinity'] }
      server_group1: "c489e94f-0c1a-40c1-a979-a897fcc6db88"

#### VMs

Now you are ready to create the VMs:

    ansible-playbook -i production infra_set_up.yml --limit=*.site2 -t os_server_all,os_data

Deploy the demo service:

    ansible-playbook -i production service_playbook.yml --limit=*.site2

#### Keep the info manually added:

If you plan to tear down your site and rebuild it, you may want to keep some stuff around that you manually set up or which would destroy precious data.
In order to do that just set the following variable to 'no' (which is the default) in the site config `<repo>/group_vars/infra.site2.yml`

    os_purge_data_volume: "no"
    os_purge_server_group: "no"
    os_purge_security_group: "no"
    os_purge_vip: "no"

Also make sure to add the floating_ip to your VM definitions.  


Clean Up
=========

Once you are done testing run:

    ansible-playbook -i production infra_clean_up.yml
    
This will remove all ressource you just created from your openstack projects. (provided you left the `os_purge_*` variables set to `yes`)


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

