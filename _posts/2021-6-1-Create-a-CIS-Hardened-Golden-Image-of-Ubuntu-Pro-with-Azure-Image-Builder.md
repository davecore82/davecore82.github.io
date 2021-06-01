---
layout: post
title: How to create a CIS hardened golden image of Ubuntu Pro with Azure Image Builder
---

This article shows you how you can use the Azure Image Builder, and the Azure CLI, to create a CIS hardened golden image of Ubuntu Pro 18.04 in a Shared Image Gallery, then distribute the image globally. 

It is based on the article [Preview: Create a Linux image and distribute it to a Shared Image Gallery by using Azure CLI](https://docs.microsoft.com/en-gb/azure/virtual-machines/linux/image-builder) in the Microsoft documentation. 

We will be using a sample .json template to configure the image. To distribute the image to a Shared Image Gallery, the template uses sharedImage as the value for the distribute section of the template.

Azure Image Builder is currently in public preview. To use Azure Image Builder during the preview, you need to register the new feature. Follow the steps in the section [Register the features](https://docs.microsoft.com/en-gb/azure/virtual-machines/linux/image-builder#register-the-features) before proceeding.

## Set variables and permissions

We will be using some pieces of information repeatedly, so we will create some variables to store that information.

For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image. Update the resource group name in this example to be the same resource group as your source managed image.

```console
# Resource group name - we are using ibLinuxGalleryRG in this example
sigResourceGroup=ibLinuxGalleryRG
# Datacenter location - we are using West US 2 in this example
location=westus2
# Additional region to replicate the image to - we are using East US in this example
additionalregion=eastus
# name of the shared image gallery - in this example we are using myGallery
sigName=myIbGallery
# name of the image definition to be created - in this example we are using myImageDef
imageDefName=myIbImageDef
# image distribution metadata reference name
runOutputName=aibLinuxSIG
```

Create a variable for your subscription ID. You can get this using `az account show -o json | grep id`

```console
subscriptionID=<Subscription ID>
```
Create the resource group.

```console
az group create -n $sigResourceGroup -l $location
```

## Create a user-assigned identity and set permissions on the resource group
Image Builder will use the user-identity provided to inject the image into the Azure Shared Image Gallery (SIG). In this example, you will create an Azure role definition that has the granular actions to perform distributing the image to the SIG. The role definition will then be assigned to the user-identity.

```console
# create user assigned identity for image builder to access the storage account where the script is located
identityName=aibBuiUserId$(date +'%s')
az identity create -g $sigResourceGroup -n $identityName

# get identity id
imgBuilderCliId=$(az identity show -g $sigResourceGroup -n $identityName -o json | grep "clientId" | cut -c16- | tr -d '",')

# get the user identity URI, needed for the template
imgBuilderId=/subscriptions/$subscriptionID/resourcegroups/$sigResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$identityName

# this command will download an Azure role definition template, and update the template with the parameters specified earlier.
curl https://raw.githubusercontent.com/Azure/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json -o aibRoleImageCreation.json

imageRoleDefName="Azure Image Builder Image Def"$(date +'%s')

# update the definition
sed -i -e "s/<subscriptionID>/$subscriptionID/g" aibRoleImageCreation.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" aibRoleImageCreation.json
sed -i -e "s/Azure Image Builder Service Image Creation Role/$imageRoleDefName/g" aibRoleImageCreation.json

# create role definitions
az role definition create --role-definition ./aibRoleImageCreation.json

# need to wait a bit here
sleep 30

# grant role definition to the user assigned identity
az role assignment create \
    --assignee $imgBuilderCliId \
    --role "$imageRoleDefName" \
    --scope /subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup
```

## Create an image definition and gallery

To use Image Builder with a shared image gallery, you need to have an existing image gallery and image definition. Image Builder will not create the image gallery and image definition for you.

If you don't already have a gallery and image definition to use, start by creating them. First, create an image gallery.

```console
az sig create \
    -g $sigResourceGroup \
    --gallery-name $sigName
```
Then, create an image definition. **Here we need to specify the plan information for Ubuntu Pro 18.04.** 

```console
az sig image-definition create \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --publisher myIbPublisher \
   --offer myOffer \
   --sku pro-18_04-lts \
   --os-type Linux \
   --plan-name pro-18_04-lts \
   --plan-product 0001-com-ubuntu-pro-bionic \
   --plan-publisher canonical
```

## Download and configure the .json
Download the .json template and configure it with your variables.

```console
curl https://raw.githubusercontent.com/Azure/azvmimagebuilder/master/quickquickstarts/1_Creating_a_Custom_Linux_Shared_Image_Gallery_Image/helloImageTemplateforSIG.json -o helloImageTemplateforSIG.json
sed -i -e "s/<subscriptionID>/$subscriptionID/g" helloImageTemplateforSIG.json
sed -i -e "s/<rgName>/$sigResourceGroup/g" helloImageTemplateforSIG.json
sed -i -e "s/<imageDefName>/$imageDefName/g" helloImageTemplateforSIG.json
sed -i -e "s/<sharedImageGalName>/$sigName/g" helloImageTemplateforSIG.json
sed -i -e "s/<region1>/$location/g" helloImageTemplateforSIG.json
sed -i -e "s/<region2>/$additionalregion/g" helloImageTemplateforSIG.json
sed -i -e "s/<runOutputName>/$runOutputName/g" helloImageTemplateforSIG.json
sed -i -e "s%<imgBuilderId>%$imgBuilderId%g" helloImageTemplateforSIG.json
```

This is where we configure the template to our needs. In this example we'll add the plan information for Ubuntu Pro 18.04 and this is where we'll also add the CIS hardening using Canonical's CIS tooling included in Ubuntu Pro. 

First, add the plan info to the source JSON block:

```json
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "0001-com-ubuntu-pro-bionic",
                "sku": "pro-18_04-lts",
                "version": "latest",
		"planInfo": {
                    "planName": "pro-18_04-lts",
                    "planProduct": "0001-com-ubuntu-pro-bionic",
                    "planPublisher": "canonical"
                }
        },
```

And then modify the customize JSON block to do the hardening:

```json
        "customize": [
            {
            "type": "Shell",
            "name": "WaitForUAtokenAutoAttach",
            "inline": [
                "sudo ua status --wait"
            ]
        },

        {
            "type": "Shell",
            "name": "EnableCISfeature",
            "inline": [
            	"sudo ua enable cis --beta"
            ]
        },

        {
            "type": "Shell",
            "name": "RunCIShardening",
            "inline": [
                "sudo /usr/share/ubuntu-scap-security-guides/cis-hardening/Canonical_Ubuntu_18.04_CIS-harden.sh lvl1_server"
            ]
        },

        {
            "type": "Shell",
            "name": "UDFworkaroundForAzureVMbooting",
            "inline": [
                "sudo rm -f /etc/modprobe.d/Canonical_Ubuntu_CIS_rule-1.1.1.7.conf"
            ]
        },

	{
            "type": "Shell",
            "name": "DetachUA",
            "inline": [
                "sudo ua detach --assume-yes && sudo rm -rf /var/log/ubuntu-advantage.log"
            ]
     	}

        ],
```
In the above example, we do the following:

 1. Wait for the UA token to auto-attach
 2. Enable the CIS feature included in Ubuntu Pro (currently in beta)
 3. Run the actual CIS hardening level 1 server script from Canonical
 4. Remove the UDF rule. This is required otherwise instances won't be able to boot in Azure. See [General Linux Installation Notes](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/create-upload-generic#general-linux-installation-notes) for details. 
 5. Detach the UA token

If we were doing a Packer template, we'd have to deprovision the Azure agent (ie. `waagent -force -deprovision`), but Azure Image Builder takes care of that for us.

This is what the full JSON AIB template should look like:

```json
{
    "type": "Microsoft.VirtualMachineImages",
    "apiVersion": "2020-02-14",
    "location": "westus2",
    "dependsOn": [],
    "tags": {
        "imagebuilderTemplate": "AzureImageBuilderSIG",
        "userIdentity": "enabled"
            },
        "identity": {
            "type": "UserAssigned",
                    "userAssignedIdentities": {
                    "/subscriptions/<your subscription ID>/resourcegroups/ibLinuxGalleryRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<your previously created user ID>": {}
                        
                }
                },
    
    "properties": {

        "buildTimeoutInMinutes" : 80,

        "vmProfile": 
            {
            "vmSize": "Standard_D1_v2",
            "osDiskSizeGB": 30
            },
        
        "source": {
            "type": "PlatformImage",
                "publisher": "Canonical",
                "offer": "0001-com-ubuntu-pro-bionic",
                "sku": "pro-18_04-lts",
                "version": "latest",
		"planInfo": {
                    "planName": "pro-18_04-lts",
                    "planProduct": "0001-com-ubuntu-pro-bionic",
                    "planPublisher": "canonical"
                }
        },
        "customize": [
            {
            "type": "Shell",
            "name": "WaitForUAtokenAutoAttach",
            "inline": [
                "sudo ua status --wait"
            ]
        },

        {
            "type": "Shell",
            "name": "EnableCISfeature",
            "inline": [
            	"sudo ua enable cis --beta"
            ]
        },

        {
            "type": "Shell",
            "name": "RunCIShardening",
            "inline": [
                "sudo /usr/share/ubuntu-scap-security-guides/cis-hardening/Canonical_Ubuntu_18.04_CIS-harden.sh lvl1_server"
            ]
        },

        {
            "type": "Shell",
            "name": "UDFworkaroundForAzureVMbooting",
            "inline": [
                "sudo rm -f /etc/modprobe.d/Canonical_Ubuntu_CIS_rule-1.1.1.7.conf"
            ]
        },

	{
            "type": "Shell",
            "name": "DetachUA",
            "inline": [
                "sudo ua detach --assume-yes && sudo rm -rf /var/log/ubuntu-advantage.log"
            ]
     	}

        ],
        "distribute": 
        [
            {   
                "type": "SharedImage",
                "galleryImageId": "/subscriptions/<your subscription ID>/resourceGroups/ibLinuxGalleryRG/providers/Microsoft.Compute/galleries/myIbGallery/images/myIbImageDef",
                "runOutputName": "aibLinuxSIG",
                "artifactTags": {
                    "source": "azureVmImageBuilder",
                    "baseosimg": "ubuntu1804"
                },
                "replicationRegions": [
                  "westus2",
                  "eastus"
                ]
            }
        ]
    }
}
```

## Create the image version

This next part will create the image version in the gallery.

Submit the image configuration to the Azure Image Builder service.

```console
az resource create \
    --resource-group $sigResourceGroup \
    --properties @helloImageTemplateforSIG.json \
    --is-full-object \
    --resource-type Microsoft.VirtualMachineImages/imageTemplates \
    -n helloImageTemplateforSIG01
```

Start the image build. **This step can take many minutes as Azure will actually launch an instance and perform the CIS hardening.** Creating the image and replicating it to both regions can take a while. Wait until this part is finished before moving on to creating a VM.

##

```console
az resource invoke-action \
     --resource-group $sigResourceGroup \
     --resource-type  Microsoft.VirtualMachineImages/imageTemplates \
     -n helloImageTemplateforSIG01 \
     --action Run
```

It's possible to see the logs of the AIB build process by going to the storage account inside the resource group created by AIB (ie. Azure Portal > Resource groups > IT_ibLinuxGalleryRG_helloImageTemplateforSI_randomID > Random ID of the storage account > Containers > packerlogs > Random ID of the container > customization.log > Download.

You should be able to see traces of the CIS hardening like this:

```
...
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Ensure no unowned files or directories exist
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Execute rule 6.1.12
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm:
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Ensure no ungrouped files or directories exist
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Execute rule 6.1.13
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm:
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Audit SUID executables
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Audit SUID executables - requires manual configuration
[7c76dfdb-9e98-4b61-9359-0638f1e04261] PACKER OUT     azure-arm: Execute rule 6.1.14
...
```
After a while, you should see the following output:

```json
{- Finished ..
  "endTime": "2021-06-01T16:45:57.207198Z",
  "name": "A6892A5A-E2BB-4D5E-B193-93A651FC4167",
  "startTime": "2021-06-01T16:09:40.05Z",
  "status": "Succeeded"
}
```

## Create the VM

Create a VM from the image version that was created by Azure Image Builder. **Since we created the image with plan information, we also need to specify it when launching the instance.**

**CAUTION:** In the below example I use my own SSH public key from my laptop. You can use yours or you can use `--generate-ssh-keys` instead of `--ssh-key-values` if you want Azure to generate an SSH key for you. Keep in mind that using `--generate-ssh-keys` may overwrite the ssh keypair “id_rsa” and “id_rsa.pub” under .ssh in your home directory.

```console
az vm create \
  --resource-group $sigResourceGroup \
  --name myAibGalleryVM \
  --admin-username aibuser \
  --location $location \
  --image "/subscriptions/$subscriptionID/resourceGroups/$sigResourceGroup/providers/Microsoft.Compute/galleries/$sigName/images/$imageDefName/versions/latest" \
  --ssh-key-values <path to your id_rsa.pub> \
  --plan-name pro-18_04-lts \
  --plan-product 0001-com-ubuntu-pro-bionic \
  --plan-publisher canonical
```

You should see this output:

```json
{- Finished ..
  "fqdns": "",
  "id": "/subscriptions/<your subscription ID>/resourceGroups/ibLinuxGalleryRG/providers/Microsoft.Compute/virtualMachines/myAibGalleryVM",
  "location": "westus2",
  "macAddress": "00-0D-3A-C4-35-DC",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.66.224.118",
  "resourceGroup": "ibLinuxGalleryRG",
  "zones": ""
}

```

SSH into the VM.

```console
ssh aibuser@<publicIpAddress>
```

You can check that the UA token is attached and the UA features are enabled:

```console
aibuser@myAibGalleryVM:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)
fips          yes       disabled  NIST-certified FIPS modules
fips-updates  yes       disabled  Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <removed>
           Subscription: <removed>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

And you can confirm that the CIS hardening is in place:

```console
aibuser@myAibGalleryVM:~$ cat /etc/issue.net
Authorized uses only. All activity may be monitored and reported.
```

You can even run a CIS score benchmark if you want:

```console
aibuser@myAibGalleryVM:~$ sudo ua enable cis --beta
One moment, checking your subscription first
Updating package lists
Installing CIS Audit packages
CIS Audit enabled

aibuser@myAibGalleryVM:~$ sudo cis-audit level1_server
Title   Ensure mounting of cramfs filesystems is disabled
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-1.1.1.1
Result  pass

Title   Ensure mounting of freevxfs filesystems is disabled
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-1.1.1.2
Result  pass

Title   Ensure mounting of jffs2 filesystems is disabled
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-1.1.1.3
Result  pass

[...]
Title   Ensure no duplicate user names exist
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-6.2.18
Result  pass

Title   Ensure no duplicate group names exist
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-6.2.19
Result  pass

Title   Ensure shadow group is empty
Rule    xccdf_com.ubuntu.bionic.cis_rule_CIS-6.2.20
Result  pass

CIS audit scan completed. The scan results are available in /usr/share/ubuntu-scap-security-guides/cis-18.04-report.html report.
```

You can download and view that HTML report. See my other articles for more details:

 - [CIS scoring comparison of the Ubuntu 16.04, 18.04 and 20.04 AMIs from CIS compared to Ubuntu Pro on AWS](https://davecore82.github.io/CIS-scoring-comparison-of-Ubuntu-from-CIS-compared-to-Ubuntu-Pro-on-AWS/)
 - [CIS scoring comparison of the Ubuntu 16.04 AMI from CIS compared to the Ubuntu Pro 16.04 AMI on the AWS Marketplace](https://davecore82.github.io/CIS-scoring-comparison-of-the-Ubuntu-16.04-AMI-from-CIS-compared-to-the-Ubuntu-Pro-16.04-AMI-on-the-AWS-Marketplace/)

## Conclusion

We've seen how you can create an Azure Image Builder template for Ubuntu Pro golden images. Feel free to customize this template to fit your needs. Users could even take further action and configure extra rules in the ruleset.params file to fix some of those failed tests (ie. Boot loader passwd, single user login, ssh access is limited, etc.).

Here are some more articles from my blog to get you going in the right direction with Ubuntu Pro:

 - [How to build an Ubuntu Pro golden image in GCP with Packer](https://davecore82.github.io/How-to-build-an-Ubuntu-Pro-golden-image-in-GCP-with-Packer/)
 - [How to build an Ubuntu Pro golden image in AWS with Packer](https://davecore82.github.io/How-to-build-an-Ubuntu-Pro-golden-image-in-AWS-with-Packer/)
 - [How to use Ubuntu Pro programmatically in AWS and Azure](https://davecore82.github.io/How-to-use-Ubuntu-Pro-programmatically-in-AWS-and-Azure/)
 - [How to get ESM in Auto Scaling groups in AWS with Ubuntu Pro 16.04](https://davecore82.github.io/How-to-get-ESM-in-Auto-Scaling-groups-in-AWS-with-Ubuntu-Pro-16.04/)
 - [How to use Azure Pipelines with a self-hosted agent running Ubuntu Pro 18.04](https://davecore82.github.io/How-to-use-Azure-Pipelines-with-a-self-hosted-agent-running-Ubuntu-Pro-18.04/)
 - [Create an Ubuntu Pro VM with infrastructure in Azure using Terraform](https://davecore82.github.io/Create-an-Ubuntu-Pro-VM-with-infrastructure-in-Azure-using-Terraform/)

