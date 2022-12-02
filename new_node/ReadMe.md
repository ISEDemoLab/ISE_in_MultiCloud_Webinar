# ISE_in_MultiCloud_Webinar `new_node/`

This folder is composed of the playbooks used to provision certain settings on any newly created node prior to joining the deployment.  The playbooks contained in this folder are set to use the `ise_nutanix` node, but you can use these playbooks for any node in your inventory.


|Playbook|Function|
|---|---|
|new_node/01-enable_api.yaml|Enable ISE ERS API and Open API|
|new_node/02-password_reset.yaml|Change ISE GUI password|
|new_node/03-PubkeyAuthentication.yaml|Create repository and enable `PubkeyAuthentication` for on-prem ISE VMs|
|new_node/04-certificates.yaml|Install Public Certificate and Root CA Certificate onto ISE|
|new_node/main.yaml|Import and run all playbooks in this folder sequentially|
