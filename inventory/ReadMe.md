# ISE_in_MultiCloud_Webinar Inventory Files

There are two default groups:
- all: contains every host
 - ungrouped: contains all hosts not belonging to another group


You may create groups based on
- What : an application, stack or microservice
- Where : a datacenter, region, local, storage, etc.
- When : the development stage for prod, test, etc.

For details on creating Ansible inventory files, see
https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html
For common patterns for targeting inventory hosts and groups:
https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html

## Dynamic Inventory

|Platform|Required filename|plugin used|
|---|---|---|
|AWS|`<name>.`aws_ec2.yaml|amazon.aws.aws_ec2|
|Azure|`<name>.`azure_rm.yaml|azure.azcollection.azure_rm|
|OCI|`<name>.`oci.yaml|oracle.oci.oci|
|VMware|`<name>.`vmware.yaml|community.vmware.vmware_vm_inventory|

The important part of the inventory filenames is that they _**END**_ with the required part of the name.  the `<name>.` portion can be omitted and was done for `aws_ec2.yaml` and `azure_rm.yaml`. The filename extension can be either `.yaml` or `.yml` which is why it is sometimes noted as `.y[a]ml`.

### `keyed_groups`
`keyed_groups` are used to assign the inventory in this platform to an `ansible-inventory` group.  These groups are shown with the command `ansible-inventory --graph` and can be a good way to keep track of certain aspects of your deployment.  There are three entries that can be used for each keyed group:
- key
- prefix
- separator

The **key** is the attribute that is used to specify the group.  In the example below, the OCI freeform tag `project` is used as the key.
The **prefix** is just that.  it's a string that is added to the beginning of the group named for the tag.  In this case, the prefix is `project`.
The **separator** is the character that srparates the prefix from the key as a visual break for the data.  In this case, the `_` is used.
```
- key: freeform_tags.project
  prefix: project
  separator: "_"
```

The Tag that was used in OCI is project:ISE_in_MultiCloud_Webinar, which returns a result of
```
  |--@project_ISE_in_MultiCloud_Webinar:
  |  |--ise-aws
  |  |--ise-azure
  |  |--ise-oci
```
in the output of `ansible-inventory --graph`.  As you can see the inventory hostname is returned from OCI under the heading that was specified in the `keyed_groups` section of the inventory file.

### `groups`
You can use `groups` (`conditional_groups` in Azure) in a similar way to assign inventory hostnames to groups.  Use the `ansible-inventory --list` command to view the attributes for the host you are trying to group.  Using this information, you can find one or more attributes that can be used as an identifying key for the system you are using.  In this case, the OCID of the installation image is used.  The `groups` section uses Jinja2 expressions to match upon.  
```
groups:
  ISE: "'ocid1.image.oc1..aaaaaaaamy3bvcmif27q4x2fd57e4hmh7hfopnesumdjxakonoo7a4uy' in image_id"
```
The expression above returns
```
  |--@ISE:
  |  |--ise-oci
```

### `compose`
The `compose` section tell Ansible how to connect to the host.  Since DNS is such a major requirement for traffic on port 443 (and port 22 for that matter), connecting to ise-oci would return an error.  The sample below shows how to take the dynamic information gathered and generate a FQDN that can be used to connect.
```
compose:
  ansible_host: hostname_label+'.securitydemo.net'
```
This way, any host that is in OCI will have .securitydemo.net added to the end of its address.  This is true with all other dynamic inventory files, but the attributes used are different for each.

### group_vars files
The files in the group_vars folder instruct Ansible on how to connect to the hosts in Dynamic Inventory.  If specifying per platform, the filenames must be the same as the Dynamic Inventory filenames.  You can also use Group Names, hence the `ISE.yaml` file.  This will instruct Ansible on how to connect to _all_ Dynamic Inventory hosts, since they are all part of the ISE group.

## Static Inventory
The Static Inventory file, `localhost.yaml` is just that.  A file where each host is input and grouped however the author would like.  There is a certain structure that must be followed, but it's a simple file to create.  Updates are manual, so when hosts are added to the network, they must be added to this file.

The file structure is:
```
---
all:
  #
  # Ungrouped Hosts - names should be DNS-compatible
  #
  hosts:
    localhost:
	    # deploy playbook to control machine, Since I am running all commands from my local PC and not
		  # using a dedicated Ansible Control Node, this is set to `local`
      ansible_connection: local 
  children:
    ise_nodes:
      hosts:
        ise-hyperv: 
          ansible_host: <IP address or FQDN> # variables cannot be used in any host section
        # # ▼ You can add more ISE nodes here ▼
        ise-kvm:
          ansible_host: <IP address or FQDN>
        ise-nutanix:
          ansible_host: <IP address or FQDN>
		  
      vars: # these group_vars are shared among the ISE nodes
        ansible_become: no # ISE does not have a superuser/enable mode
        ansible_connection: ansible.netcommon.network_cli
        ansible_network_os: cisco.ios.ios
        # ⭐ ISE SSH username is `iseadmin` for ISE 3.2+ cloud instances!
        ansible_ssh_user: "{{ lookup('env','ISE_SSH_USERNAME') }}"
        # ⭐ Un/Comment the SSH password or private_key_file - whichever you use!
        # ansible_ssh_password: "{{ lookup('env','ISE_REST_PASSWORD') }}"
        # ansible_ssh_password: "{{ lookup('env','ISE_INIT_PASSWORD') }}"
        ansible_ssh_private_key_file: "{{ ssh_key_private_file }}"

  # global vars
  vars: #add global variables here.  There is a list of global variables in the `localhost.yaml` file.
```

The ISE nodes in this inventory file are grouped into the `ise_nodes` group and will be returned from the `ansible-inventory --graph` command as:
```
  |--@ise_nodes:
  |  |--ise-hyperv
  |  |--ise-kvm
  |  |--ise-nutanix
```

## License

MIT

## Author

Charlie Moreton, <https://github.com/ISEDemoLab>