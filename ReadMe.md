# ISE_in_MultiCloud_Webinar

This repository is used for the ISE in a Hybrid Cloud Environment webinar that was given on December 6, 2022.  The purpose of the webinar was to show ISE deployed on all supported virtual environments (including public clouds) as part of a single deployment.

![ISE in MultiCloud Topology](https://github.com/ISEDemoLab/ISE_in_MultiCloud_Webinar/blob/main/images/ISE_in_MultiCloud_Webinar.png)

The Ansible playbooks in this repository are listed in this table with their functions:
|Playbook|Function|
|---|---|
|inventory/azure_rm.yaml, inventory/aws_ec2.yaml, inventory/ise.oci.yaml, inventory/demo.vmware.yaml|Gather dynamic inventory from the three different public clouds (Azure, AWS, OCI) and VMware|
|inventory/localhost.yaml|Track inventory for on-prem instances|
|new_node/01-enable_api.yaml|Enable ISE ERS API and Open API|
|new_node/02-password_reset.yaml|Change ISE GUI password|
|new_node/03-PubkeyAuthentication.yaml|Create repository and enable `PubkeyAuthentication` for on-prem ISE VMs|
|new_node/04-certificates.yaml|Install Public Certificate and Root CA Certificate onto ISE|
|01-show_version.yaml|Show ISE version based on inventory files|
|02-azure_deploy.yaml|Create a new Resource Group in Azure (including VN, subnets, etc.), deploy ISE, and create a VPN connection with a route table|
|03-azure_existing_rg.yaml|Deploy ISE in an existing Resource Group in Azure|
|04-vm_deploy.yaml|Deploy ISE on VMware ESXi using ZTP (ESXi 7.0 U3 is used in this lab)|
|05-vm_power_on.yaml|Power on the VMware VM to show the ZTP process|
|06-standalone_to_primary.yaml|Convert a Standalone node to a Primary node|
|07-add_initial_nodes.yaml|Register the SPAN, PMnT, and PSN to the deployment|
|08-remove_ppan_services.yaml|Remove the SMnT Role and all services from the PPAN|
|09-add_nodes.yaml|Register the SMnT and the last 3 PSNs to the deployment|
|10-create_node_groups.yaml|Create Node Groups and add nodes|
|11-azure_destroy.yaml|Destroy Azure instance and Resource Group|
|12-vm_destroy.yaml|Destroy VMware instance|
|13-remove_nodes|Delete Node Groups, Deregister SMnT and 3 PSNs, Add SMnT and services to PPAN (post-Playbook 7 status)|

 To use Zero Touch Provisioning (ZTP) with VMware, you need to have an understanding of this article:  [ISE Zero Touch Provisioning (ZTP)](https://community.cisco.com/t5/security-knowledge-base/ise-zero-touch-provisioning-ztp/ta-p/4541606)

## Quick Start

Clone this repository:  

```sh
git clone https://github.com/ISEDemoLab/ISE_in_MultiCloud_Webinar.git
```

Create your Python environment and install Ansible:
> ⚠ Installing Ansible using Linux packages (`sudo apt install ansible`) may result in a much older version of Ansible being installed.
> 💡 Installing Ansible with Python packages will get you the latest.
> 💡 If you have any problems installing Python or Ansible, see [Installing Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).

```sh
pipenv install --python 3.9                # use Python 3.9 or later
pipenv install paramiko                    # ISE SSH/CLI access
pipenv install ansible                     # Ansible packages
pipenv install ciscoisesdk jmespath        # ISE and friends
pipenv install ansible[azure]              # Python packages for Azure 
pipenv install boto3 botocore              # Python packages for AWS 
pipenv install oci                         # Python packages for OCI
pipenv shell                               # Launch the virtual environment
```

Download the Azure python library requirements doc and install the packages listed in the document:

```
wget -nv -q https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
pipenv install -r requirements-azure.txt 
```

The `azure.storage.cloudstorageaccount` module is not included in the Ansible collection by default, so install it with this command:

```
pipenv install azure-storage==0.35.1 
```

## Requirements
### Create an AWS IAM user for API and create an Access key under that user:
[How do I create an AWS access key?](https://aws.amazon.com/premiumsupport/knowledge-center/create-access-key/)
This is a knowledge base article that contains links to the detailed steps needed.

### Create an Azure service principal using the Azure CLI:
Use this page to create the Azure service principal:
[Quickstart: Create an Azure service principal for Ansible](https://learn.microsoft.com/en-us/azure/developer/ansible/create-ansible-service-principal?tabs=azure-cli)
> ⚠ There is a typo in the **Assign a role to the Azure service principal** section.  
> 💡 The text reads **"Replace `<appID>` with the value provided from the output of `az ad sp create-for-rba` command."**

> 💡 It should read **"Replace `<appID>` with the value provided from the output of `az ad sp create-for-rbac` command."** which is the command from the first step.

### Create an API Signing Key and default config file for OCI:
[SDK and CLI Configuration File](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm#SDK_and_CLI_Configuration_File)
[Required Keys and OCIDs](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm)

### Configure ESXi API for Ansible
[VMware Prerequisites](https://docs.ansible.com/ansible/latest/collections/community/vmware/docsite/vmware_scenarios/vmware_requirements.html)

### Install the VMware vSphere Automation SDK for Python for `community.vmware` collection modules
[VMware vSphere Automation SDK for Python](https://medium.com/@hegdeanusha25/getting-started-with-the-vmware-vsphere-automation-sdk-for-python-505b9c4b03c9)
If your virtual environment is already setup, just run the following command:
```sh
pip install — upgrade git+https://github.com/vmware/vsphere-automation-sdk-python.git
```
This is only needed for the dynamic inventory in this lab.  

### The following Ansible modules are needed:

- azure.azcollection
- amazon.aws
- oracle.oci
- vmware.vmware_rest
- community.vmware
- cisco.ise
- cisco.ios
- ansible.netcommon

Export your Azure and ISE credentials into your terminal environment:  

```sh
export AZURE_CLIENT_ID=<service_principal_client_id>
export AZURE_SECRET=<service_principal_password>
export AZURE_SUBSCRIPTION_ID=<azure_subscription_id>
export AZURE_TENANT=<azure_tenant_id>

export AWS_REGION=<region>
export AWS_ACCESS_KEY=<access_key>
export AWS_SECRET_KEY=<secret_key>

export VCENTER=<hostname_or_ip_address>
export VCENTER_USERNAME=<vcenter_username>
export VCENTER_PASSWORD=<vcenter_password>
export VMWARE_HOST=<hostname_or_ip_address>
export VMWARE_USER=<esxi_username>
export VMWARE_PASSWORD=<esxi_password>

export ISE_USERNAME=iseadmin
export ISE_PASSWORD=<ise_password>
export ISE_VERIFY=False
export ISE_DEBUG=False
```

or you may edit and source these variables from a file in your `~/.secrets` directory.  I keep my credentials for each platform in a separate file, this way I can source only the credentials needed for the current project.  OCI doesn't require this since it sources the credentials from the default config file as detailed above.

```sh
source ~/.secrets/azure.sh
source ~/.secrets/aws.sh
source ~/.secrets/vmware.sh
source ~/.secrets/ise_cli.sh
```

## Variables

Edit the project and deployment settings in `vars/main.yaml` to match your environment:

```yaml
project_name: ISE_in_MultiCloud_Webinar
```

Run an Ansible playbook:  

```sh
ansible-playbook playbook.yaml
```

## Example Playbook

### Run the playbook to install certificates onto the `ise-nutanix` node:
```sh
ansible-playbook new_node/04-certificates.yaml
```

Contents of `new_node/04-certificates.yaml`:
```new_node/04-certificates.yaml
---
- name: Import certificates into ISE
  gather_facts: no
  hosts: localhost
  # connection: local
  vars_files:
    - ~/ISE_in_Azure/vars/main.yaml

  tasks:
    #----------------------------------------------------------------------------
    # 🛈 The `data`and `privateKeyData` fields require the plain-text 
    # contents of the certificate file. Every space needs to be 
    # replaced with a newline escape sequence (\n) character.
    #
    # In your *nix terminal, use the command
    # awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' <<your pem file>>
    # to automatically format your .pem certificates and .pvk 
    # private key files to be read by ISE.
    #
    # ⚠ The CA Cert chain MUST be imported before you can import the 
    # system certificate.
    #----------------------------------------------------------------------------

    - name: Import Hydrant CA Cert to Trusted Cert Store
      cisco.ise.trusted_certificate_import:
        ise_hostname: "{{ ise_nutanix }}"
        ise_username: "{{ ise_username }}"
        ise_password: "{{ ise_password }}"
        ise_verify: "{{ ise_verify }}"
        allowBasicConstraintCAFalse: true
        allowOutOfDateCert: true
        allowSHA1Certificates: true
        data: "{{ ca_cert }}"
        description: string
        name: HydrantID Server CA O1
        trustForCertificateBasedAdminAuth: true
        trustForCiscoServicesAuth: true
        trustForClientAuth: true
        trustForIseAuth: true
        validateCertificateExtensions: true

    
    - name: Import securitydemo.net system certificate
      cisco.ise.system_certificate_import:
        ise_hostname: "{{ ise_nutanix }}"
        ise_username: "{{ ise_username }}"
        ise_password: "{{ ise_password }}"
        ise_verify: "{{ ise_verify }}"
        admin: true
        allowExtendedValidity: true
        allowOutOfDateCert: true
        allowPortalTagTransferForSameSubject: true
        allowReplacementOfCertificates: true
        allowReplacementOfPortalGroupTag: true
        allowRoleTransferForSameSubject: true
        allowSHA1Certificates: true
        allowWildCardCertificates: true
        data: "{{ certificate }}"
        eap: false
        ims: false
        name: securitydemo.net
        password: "{{ pvk_password }}"
        portal: true
        portalGroupTag: "Default Portal Certificate Group"
        privateKeyData: "{{ private_key }}"
        pxgrid: false
        radius: false
        saml: false
        validateCertificateExtensions: true
```

### Run multiple playbooks sequentially:
```sh
ansible-playbook new_node/main.yaml
```

Contents of `new_node/main.yaml`.  This will enable APIs, PubkeyAuthentication, and install certificates, but will skip the password reset as it is commented out.  Each playbook will finish before moving on to the next.
```new_node/main.yaml
---
- name: Enable ISE APIs
  import_playbook: 01-enable_api.yaml

# - name: Reset GUI Password
#   import_playbook: 02-password_reset.yaml

- name: Enable SSH Public Key Authentication
  import_playbook: 03-PubkeyAuthentication.yaml

- name: Install Trusted and System Certificates
  import_playbook: 04-certificates.yaml
  ```

## License

MIT

## Author

Charlie Moreton, <https://github.com/ISEDemoLab>
