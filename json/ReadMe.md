# ISE_in_MultiCloud_Webinar .json template files
This folder contains _sanitized_ JSON templates for the following Azure Resource Manager deployments used in this lab:

|Template Name|Parameters File|Function|
|---|---|---|
|azuredeploy.json|azuredeploy.parameters.json|Deploy ISE 3.2 **Virtual Machine** from the Marketplace with a nic, private ip and public ip|
|azure_lng.json|azure_lng.parameters.json|Create a Local Network Gateway|
|azure_vpn.json|azure_vpn.parameters.json|Create a VPN connection from the Virtual Network Gateway to the Local Network Gateway|
|azure_existing_rg.json|azure_existing_rg.parameters.json|Deploy ISE 3.2 **Virtual Machine** from Marketplace to an existing RG with an existing storage account|

The first three rows in the table are used in the `02-azure_deploy.yaml` playbook, while the last row is used in the `03-azure_existing_rg_deploy.yaml` playbook.  The template files are just as they would be downloaded from Azure, the parameters files will need to be updated with your information.  I have removed the subscription id, ssh key, and user data entries from all parameters files.

These templates are used due to the fact that 
- There is no Ansible module that allows for the User Data to be collected and used for the ISE deployment, which is required
- There is no Ansible module to create a Local Network Gateway or a VPN connection

While I am grateful that Azure allows for the templates as a workaround to these shortcomings, I would hope that they add modules to their collection that address them.  The templates add a LOT of unnecessary steps to the process.

If you would like to create the ISE deployment from the first row of the table, but do not want a public IP address, delete the following lines in the parameters file and the corresponding lines in the template file. The lines to delete are marked with `//` in the text below.
```
      // "publicIpAddressName": {
      //   "value": "ise32-d4s-ip"
      // },
      // "publicIpAddressType": {
      //   "value": "Static"
      // },
      // "publicIpAddressSku": {
      //   "value": "Standard"
      // },
      // "pipDeleteOption": {
      //   "value": "Delete"
      // },
```
Also delete the following lines in the template file (The lines to delete are marked with `//` in the text below.):
```
    "resources": [
        {
            "name": "[parameters('networkInterfaceName')]",
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2021-03-01",
            "location": "[parameters('location')]",
//            "dependsOn": [
//                "[concat('Microsoft.Network/publicIpAddresses/', parameters('publicIpAddressName'))]"
//            ],
            "properties": {
                "ipConfigurations": [
                    {
                        "name": "ipconfig1",
                        "properties": {
                            "subnet": {
                                "id": "[variables('subnetRef')]"
                            },
                            "privateIPAllocationMethod": "Static",
                            "privateIPAddress": "[parameters('privateIp')]",
//                            "publicIpAddress": {
//                                "id": "[resourceId(resourceGroup().name, 'Microsoft.Network/publicIpAddresses', parameters('publicIpAddressName'))]",
//                                "properties": {
//                                    "deleteOption": "[parameters('pipDeleteOption')]"
//                                }
//                            }
                        }
                    }
                ]
            }
        },
//        {
//            "name": "[parameters('publicIpAddressName')]",
//            "type": "Microsoft.Network/publicIpAddresses",
//            "apiVersion": "2020-08-01",
//            "location": "[parameters('location')]",
//            "properties": {
//                "publicIpAllocationMethod": "[parameters('publicIpAddressType')]"
//            },
//            "sku": {
//                "name": "[parameters('publicIpAddressSku')]"
//            }
//        },
```



## License

MIT

## Author

Charlie Moreton, <https://github.com/ISEDemoLab>