---
layout: post
title: Create an Ubuntu Pro VM with infrastructure in Azure using Terraform
---

Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files. This article shows you how to create an Ubuntu Pro 18.04 VM and supporting resources in Azure with Terraform. 

If you don't know already, [Ubuntu Pro](https://ubuntu.com/azure/pro) is a premium image designed by Canonical to provide additional coverage for production environments running in the cloud. It includes security and compliance services, enabled by default, suitable for small to largescale Linux enterprise operations — no contract needed.

Key features include live kernel patching, which provides instant security and longer uptimes, broad security patching of major open source workloads for production use, and certified components for FedRAMP, HIPAA, PCI and ISO use cases. Ubuntu Pro (18.04 and above) is backed by a 10-year maintenance commitment by Canonical.

TL;DR, here is what you need to change in your Terraform config to use Ubuntu Pro instead of Ubuntu LTS:

```diff
     source_image_reference {
         publisher = "Canonical"
-        offer     = "UbuntuServer"
-        sku       = "18.04-LTS"
+        offer     = "0001-com-ubuntu-pro-bionic"
+        sku       = "pro-18_04-lts"
         version   = "latest"
     }
 
+    plan {
+        name = "pro-18_04-lts"
+        product = "0001-com-ubuntu-pro-bionic"
+        publisher = "canonical"
+    }
```


There is an [excellent article in the Microsoft documentation](https://docs.microsoft.com/en-us/azure/developer/terraform/create-linux-virtual-machine-with-infrastructure) that already explains how to create a Linux VM with infrastructure in Azure using Terraform that uses a regular Ubuntu 18.04 LTS VM. I'm going to use the same example from that page but point out the few things that need to be changed to use Ubuntu Pro 18.04 instead of regular Ubuntu 18.04.


It's necessary to add the plan block because the Ubuntu Pro image comes from the Marketplace and requires a subscription. This plan block is how you do that.

The offer and the sku change too. Here's how you can see the different offers from Canonical:

```console
$ az vm image list-offers -p canonical -l eastus | grep name

    [...]
    "name": "0001-com-ubuntu-pro-bionic",
    "name": "0001-com-ubuntu-pro-bionic-fips",
    "name": "0001-com-ubuntu-pro-focal",
    "name": "0001-com-ubuntu-pro-trusty",
    "name": "0001-com-ubuntu-pro-xenial",
    "name": "0001-com-ubuntu-pro-xenial-fips",
    [...]
    "name": "UbuntuServer",
    "name": "Ubuntu_Core",
```

And then you can list the images inside one of those offers like this:

```console
$ az vm image list -p canonical -l eastus --offer 0001-com-ubuntu-pro-bionic --all | grep sku | sort -u

    "sku": "pro-18_04-lts",
    "sku": "pro-18_04-lts-gen2",
    "sku": "pro-fips-18_04",
    "sku": "pro-fips-18_04-gen2",
```

If you want you can look for the specific URN of an image in one of these skus with this command:

```console
$ az vm image list -p canonical -l eastus --offer 0001-com-ubuntu-pro-bionic --all --sku pro-18_04-lts

[
  {
    "offer": "0001-com-ubuntu-pro-bionic",
    "publisher": "Canonical",
    "sku": "pro-18_04-lts",
    "urn": "Canonical:0001-com-ubuntu-pro-bionic:pro-18_04-lts:18.04.20200318",
    "version": "18.04.20200318"
  },
[...]
  {
    "offer": "0001-com-ubuntu-pro-bionic",
    "publisher": "Canonical",
    "sku": "pro-18_04-lts-gen2",
    "urn": "Canonical:0001-com-ubuntu-pro-bionic:pro-18_04-lts-gen2:18.04.202103250",
    "version": "18.04.202103250"
  }
]
```

But for our example, we just use the latest image available in the pro-18_04-lts sku.

In my Terraform config, I also changed the admin username to ubuntu instead of azureuser and I use my own local SSH key instead of creating a new one in Azure.  After doing `terraform init`, here's what the `terraform apply` looks like:

```console
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_linux_virtual_machine.myterraformvm will be created
  + resource "azurerm_linux_virtual_machine" "myterraformvm" {
      + admin_username                  = "ubuntu"
      + allow_extension_operations      = true
      + computer_name                   = "myvm"
      + disable_password_authentication = true
      + extensions_time_budget          = "PT1H30M"
      + id                              = (known after apply)
      + location                        = "eastus"
      + max_bid_price                   = -1
      + name                            = "myVM"
      + network_interface_ids           = (known after apply)
      + platform_fault_domain           = -1
      + priority                        = "Regular"
      + private_ip_address              = (known after apply)
      + private_ip_addresses            = (known after apply)
      + provision_vm_agent              = true
      + public_ip_address               = (known after apply)
      + public_ip_addresses             = (known after apply)
      + resource_group_name             = "myResourceGroup"
      + size                            = "Standard_DS1_v2"
      + tags                            = {
          + "environment" = "Terraform Demo"
        }
      + virtual_machine_id              = (known after apply)
      + zone                            = (known after apply)

      + admin_ssh_key {
          + public_key = <<-EOT
                ssh-rsa <redacted>
            EOT
          + username   = "ubuntu"
        }

      + boot_diagnostics {
          + storage_account_uri = (known after apply)
        }

      + os_disk {
          + caching                   = "ReadWrite"
          + disk_size_gb              = (known after apply)
          + name                      = "myOsDisk"
          + storage_account_type      = "Premium_LRS"
          + write_accelerator_enabled = false
        }

      + plan {
          + name      = "pro-18_04-lts"
          + product   = "0001-com-ubuntu-pro-bionic"
          + publisher = "canonical"
        }

      + source_image_reference {
          + offer     = "0001-com-ubuntu-pro-bionic"
          + publisher = "Canonical"
          + sku       = "pro-18_04-lts"
          + version   = "latest"
        }
    }

  # azurerm_network_interface.myterraformnic will be created
  + resource "azurerm_network_interface" "myterraformnic" {
      + applied_dns_servers           = (known after apply)
      + dns_servers                   = (known after apply)
      + enable_accelerated_networking = false
      + enable_ip_forwarding          = false
      + id                            = (known after apply)
      + internal_dns_name_label       = (known after apply)
      + internal_domain_name_suffix   = (known after apply)
      + location                      = "eastus"
      + mac_address                   = (known after apply)
      + name                          = "myNIC"
      + private_ip_address            = (known after apply)
      + private_ip_addresses          = (known after apply)
      + resource_group_name           = "myResourceGroup"
      + tags                          = {
          + "environment" = "Terraform Demo"
        }
      + virtual_machine_id            = (known after apply)

      + ip_configuration {
          + name                          = "myNicConfiguration"
          + primary                       = (known after apply)
          + private_ip_address            = (known after apply)
          + private_ip_address_allocation = "dynamic"
          + private_ip_address_version    = "IPv4"
          + public_ip_address_id          = (known after apply)
          + subnet_id                     = (known after apply)
        }
    }

  # azurerm_network_interface_security_group_association.example will be created
  + resource "azurerm_network_interface_security_group_association" "example" {
      + id                        = (known after apply)
      + network_interface_id      = (known after apply)
      + network_security_group_id = (known after apply)
    }

  # azurerm_network_security_group.myterraformnsg will be created
  + resource "azurerm_network_security_group" "myterraformnsg" {
      + id                  = (known after apply)
      + location            = "eastus"
      + name                = "myNetworkSecurityGroup"
      + resource_group_name = "myResourceGroup"
      + security_rule       = [
          + {
              + access                                     = "Allow"
              + description                                = ""
              + destination_address_prefix                 = "*"
              + destination_address_prefixes               = []
              + destination_application_security_group_ids = []
              + destination_port_range                     = "22"
              + destination_port_ranges                    = []
              + direction                                  = "Inbound"
              + name                                       = "SSH"
              + priority                                   = 1001
              + protocol                                   = "Tcp"
              + source_address_prefix                      = "*"
              + source_address_prefixes                    = []
              + source_application_security_group_ids      = []
              + source_port_range                          = "*"
              + source_port_ranges                         = []
            },
        ]
      + tags                = {
          + "environment" = "Terraform Demo"
        }
    }

  # azurerm_public_ip.myterraformpublicip will be created
  + resource "azurerm_public_ip" "myterraformpublicip" {
      + allocation_method       = "Dynamic"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + idle_timeout_in_minutes = 4
      + ip_address              = (known after apply)
      + ip_version              = "IPv4"
      + location                = "eastus"
      + name                    = "myPublicIP"
      + resource_group_name     = "myResourceGroup"
      + sku                     = "Basic"
      + tags                    = {
          + "environment" = "Terraform Demo"
        }
    }

  # azurerm_resource_group.myterraformgroup will be created
  + resource "azurerm_resource_group" "myterraformgroup" {
      + id       = (known after apply)
      + location = "eastus"
      + name     = "myResourceGroup"
      + tags     = {
          + "environment" = "Terraform Demo"
        }
    }

  # azurerm_storage_account.mystorageaccount will be created
  + resource "azurerm_storage_account" "mystorageaccount" {
      + access_tier                      = (known after apply)
      + account_kind                     = "StorageV2"
      + account_replication_type         = "LRS"
      + account_tier                     = "Standard"
      + allow_blob_public_access         = false
      + enable_https_traffic_only        = true
      + id                               = (known after apply)
      + is_hns_enabled                   = false
      + large_file_share_enabled         = (known after apply)
      + location                         = "eastus"
      + min_tls_version                  = "TLS1_0"
      + name                             = (known after apply)
      + primary_access_key               = (sensitive value)
      + primary_blob_connection_string   = (sensitive value)
      + primary_blob_endpoint            = (known after apply)
      + primary_blob_host                = (known after apply)
      + primary_connection_string        = (sensitive value)
      + primary_dfs_endpoint             = (known after apply)
      + primary_dfs_host                 = (known after apply)
      + primary_file_endpoint            = (known after apply)
      + primary_file_host                = (known after apply)
      + primary_location                 = (known after apply)
      + primary_queue_endpoint           = (known after apply)
      + primary_queue_host               = (known after apply)
      + primary_table_endpoint           = (known after apply)
      + primary_table_host               = (known after apply)
      + primary_web_endpoint             = (known after apply)
      + primary_web_host                 = (known after apply)
      + resource_group_name              = "myResourceGroup"
      + secondary_access_key             = (sensitive value)
      + secondary_blob_connection_string = (sensitive value)
      + secondary_blob_endpoint          = (known after apply)
      + secondary_blob_host              = (known after apply)
      + secondary_connection_string      = (sensitive value)
      + secondary_dfs_endpoint           = (known after apply)
      + secondary_dfs_host               = (known after apply)
      + secondary_file_endpoint          = (known after apply)
      + secondary_file_host              = (known after apply)
      + secondary_location               = (known after apply)
      + secondary_queue_endpoint         = (known after apply)
      + secondary_queue_host             = (known after apply)
      + secondary_table_endpoint         = (known after apply)
      + secondary_table_host             = (known after apply)
      + secondary_web_endpoint           = (known after apply)
      + secondary_web_host               = (known after apply)
      + tags                             = {
          + "environment" = "Terraform Demo"
        }

      + blob_properties {
          + container_delete_retention_policy {
              + days = (known after apply)
            }

          + cors_rule {
              + allowed_headers    = (known after apply)
              + allowed_methods    = (known after apply)
              + allowed_origins    = (known after apply)
              + exposed_headers    = (known after apply)
              + max_age_in_seconds = (known after apply)
            }

          + delete_retention_policy {
              + days = (known after apply)
            }
        }

      + identity {
          + principal_id = (known after apply)
          + tenant_id    = (known after apply)
          + type         = (known after apply)
        }

      + network_rules {
          + bypass                     = (known after apply)
          + default_action             = (known after apply)
          + ip_rules                   = (known after apply)
          + virtual_network_subnet_ids = (known after apply)
        }

      + queue_properties {
          + cors_rule {
              + allowed_headers    = (known after apply)
              + allowed_methods    = (known after apply)
              + allowed_origins    = (known after apply)
              + exposed_headers    = (known after apply)
              + max_age_in_seconds = (known after apply)
            }

          + hour_metrics {
              + enabled               = (known after apply)
              + include_apis          = (known after apply)
              + retention_policy_days = (known after apply)
              + version               = (known after apply)
            }

          + logging {
              + delete                = (known after apply)
              + read                  = (known after apply)
              + retention_policy_days = (known after apply)
              + version               = (known after apply)
              + write                 = (known after apply)
            }

          + minute_metrics {
              + enabled               = (known after apply)
              + include_apis          = (known after apply)
              + retention_policy_days = (known after apply)
              + version               = (known after apply)
            }
        }
    }

  # azurerm_subnet.myterraformsubnet will be created
  + resource "azurerm_subnet" "myterraformsubnet" {
      + address_prefix                                 = (known after apply)
      + address_prefixes                               = [
          + "10.0.1.0/24",
        ]
      + enforce_private_link_endpoint_network_policies = false
      + enforce_private_link_service_network_policies  = false
      + id                                             = (known after apply)
      + name                                           = "mySubnet"
      + resource_group_name                            = "myResourceGroup"
      + virtual_network_name                           = "myVnet"
    }

  # azurerm_virtual_network.myterraformnetwork will be created
  + resource "azurerm_virtual_network" "myterraformnetwork" {
      + address_space         = [
          + "10.0.0.0/16",
        ]
      + guid                  = (known after apply)
      + id                    = (known after apply)
      + location              = "eastus"
      + name                  = "myVnet"
      + resource_group_name   = "myResourceGroup"
      + subnet                = (known after apply)
      + tags                  = {
          + "environment" = "Terraform Demo"
        }
      + vm_protection_enabled = false
    }

  # random_id.randomId will be created
  + resource "random_id" "randomId" {
      + b64_std     = (known after apply)
      + b64_url     = (known after apply)
      + byte_length = 8
      + dec         = (known after apply)
      + hex         = (known after apply)
      + id          = (known after apply)
      + keepers     = {
          + "resource_group" = "myResourceGroup"
        }
    }

Plan: 10 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_resource_group.myterraformgroup: Creating...
[...]
```

Once the creation completes, you can list the VMs in the ressource group:

```console
$ az vm list --resource-group myResourceGroup -o table -d
Name    ResourceGroup    PowerState    PublicIps      Fqdns    Location    Zones
------  ---------------  ------------  -------------  -------  ----------  -------
myVM    myResourceGroup  VM running    52.249.198.68           eastus
```

In my case, I can SSH in the VM with my SSH key and the ubuntu user:

```console
$ ssh ubuntu@52.249.198.68

Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 5.4.0-1043-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Thu Apr 15 19:53:08 UTC 2021

  System load:  0.45              Processes:           121
  Usage of /:   5.5% of 28.90GB   Users logged in:     0
  Memory usage: 6%                IP address for eth0: 10.0.1.4
  Swap usage:   0%

UA Infra: Extended Security Maintenance (ESM) is enabled.

36 packages can be updated.
5 of these updates are fixed through UA Infra: ESM.
17 of these updates are security updates.
To see these additional updates run: apt list --upgradable



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```
And I can view the status of the UA services that come with Ubuntu Pro:

```console
ubuntu@myvm:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)
fips          yes       disabled  NIST-certified FIPS modules
fips-updates  yes       disabled  Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <redacted>
           Subscription: <redacted>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

Don't forget to destroy your Terraform environment with `terraform destroy` if you no longer need it.

Here is my complete `config.tf` file:

```
# Configure the Microsoft Azure Provider
provider "azurerm" {
    features {}
}

# Create a resource group if it doesn't exist
resource "azurerm_resource_group" "myterraformgroup" {
    name     = "myResourceGroup"
    location = "eastus"

    tags = {
        environment = "Terraform Demo"
    }
}

# Create virtual network
resource "azurerm_virtual_network" "myterraformnetwork" {
    name                = "myVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "eastus"
    resource_group_name = azurerm_resource_group.myterraformgroup.name

    tags = {
        environment = "Terraform Demo"
    }
}

# Create subnet
resource "azurerm_subnet" "myterraformsubnet" {
    name                 = "mySubnet"
    resource_group_name  = azurerm_resource_group.myterraformgroup.name
    virtual_network_name = azurerm_virtual_network.myterraformnetwork.name
    address_prefixes       = ["10.0.1.0/24"]
}

# Create public IPs
resource "azurerm_public_ip" "myterraformpublicip" {
    name                         = "myPublicIP"
    location                     = "eastus"
    resource_group_name          = azurerm_resource_group.myterraformgroup.name
    allocation_method            = "Dynamic"

    tags = {
        environment = "Terraform Demo"
    }
}

# Create Network Security Group and rule
resource "azurerm_network_security_group" "myterraformnsg" {
    name                = "myNetworkSecurityGroup"
    location            = "eastus"
    resource_group_name = azurerm_resource_group.myterraformgroup.name

    security_rule {
        name                       = "SSH"
        priority                   = 1001
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_range     = "22"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
    }

    tags = {
        environment = "Terraform Demo"
    }
}

# Create network interface
resource "azurerm_network_interface" "myterraformnic" {
    name                      = "myNIC"
    location                  = "eastus"
    resource_group_name       = azurerm_resource_group.myterraformgroup.name

    ip_configuration {
        name                          = "myNicConfiguration"
        subnet_id                     = azurerm_subnet.myterraformsubnet.id
        private_ip_address_allocation = "Dynamic"
        public_ip_address_id          = azurerm_public_ip.myterraformpublicip.id
    }

    tags = {
        environment = "Terraform Demo"
    }
}

# Connect the security group to the network interface
resource "azurerm_network_interface_security_group_association" "example" {
    network_interface_id      = azurerm_network_interface.myterraformnic.id
    network_security_group_id = azurerm_network_security_group.myterraformnsg.id
}

# Generate random text for a unique storage account name
resource "random_id" "randomId" {
    keepers = {
        # Generate a new ID only when a new resource group is defined
        resource_group = azurerm_resource_group.myterraformgroup.name
    }

    byte_length = 8
}

# Create storage account for boot diagnostics
resource "azurerm_storage_account" "mystorageaccount" {
    name                        = "diag${random_id.randomId.hex}"
    resource_group_name         = azurerm_resource_group.myterraformgroup.name
    location                    = "eastus"
    account_tier                = "Standard"
    account_replication_type    = "LRS"

    tags = {
        environment = "Terraform Demo"
    }
}

# Create virtual machine
resource "azurerm_linux_virtual_machine" "myterraformvm" {
    name                  = "myVM"
    location              = "eastus"
    resource_group_name   = azurerm_resource_group.myterraformgroup.name
    network_interface_ids = [azurerm_network_interface.myterraformnic.id]
    size                  = "Standard_DS1_v2"

    os_disk {
        name              = "myOsDisk"
        caching           = "ReadWrite"
        storage_account_type = "Premium_LRS"
    }

    source_image_reference {
        publisher = "Canonical"
        offer     = "0001-com-ubuntu-pro-bionic"
        sku       = "pro-18_04-lts"
        version   = "latest"
    }

    # Add plan info because Ubuntu Pro comes from the marketplace
    plan {
        name = "pro-18_04-lts"
        product = "0001-com-ubuntu-pro-bionic"
        publisher = "canonical"
    }

    computer_name  = "myvm"
    admin_username = "ubuntu"
    disable_password_authentication = true

    admin_ssh_key {
        username       = "ubuntu"
	public_key = file("~/.ssh/id_rsa.pub")
    }

    boot_diagnostics {
        storage_account_uri = azurerm_storage_account.mystorageaccount.primary_blob_endpoint
    }

    tags = {
        environment = "Terraform Demo"
    }
}
```


