---
layout: post
title: How to use Ubuntu Pro programmatically in AWS and Azure
---

This article explains how to use Ubuntu Pro programmatically in AWS and Azure. The process is a bit different for each cloud and I'll explain the differences here.

In AWS, all you need to do is subscribe to the marketplace product before-hand. If you try to launch an instance with a marketplace AMI without a subscription, you'll get the following error message (this example is with Terraform trying to launch Ubuntu Pro 20.04):

```
aws_instance.instance1-pro: Creating...
╷
│ Error: Error launching source instance: OptInRequired: In order to use this AWS Marketplace product you need to accept terms and subscribe. To do so please visit https://aws.amazon.com/marketplace/pp?sku=abwe6hiebq70xss7fo2neefpy
│ 	status code: 401, request id: 0c05df3d-e306-4400-a0ca-277f2232c730
```

You can accept the terms and subscribe by visiting the link in the error message or by finding the product in the AWS Marketplace and subscribing to it.  For example, in the AWS services list, select AWS Marketplace Subscriptions. Click on Discover products to the left and search for ubuntu pro. Pick the version you need (ie. Ubuntu Pro 20.04 LTS) and on the page of that product click on Continue to Subscribe. In the Subscribe to this software page, click on Accept Terms to accept the extra pricing and terms and wait a few minutes for the subscription to complete.

Once that's done you can now launch an Ubuntu Pro 20.04 instance programatically. Here's a simple Terraform example:

```console
$ cat config.tf

provider "aws" {
  access_key = "<your access key>"
  secret_key = "<your secret key>"
  region = "us-east-1"
}

resource "aws_instance" "instance1-pro" {
  ami = "ami-0ed28656d62ce20d6"
  instance_type = "t2.small"
  key_name = "mysshkey"
  tags = {
    Name = "Ubuntu-20.04-pro"
  }

$ terraform apply

[...]
  # aws_instance.instance1-pro will be created
  + resource "aws_instance" "instance1-pro" {
      + ami                          = "ami-0ed28656d62ce20d6"
[...]

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.instance1-pro: Creating...
aws_instance.instance1-pro: Still creating... [10s elapsed]
aws_instance.instance1-pro: Still creating... [20s elapsed]
aws_instance.instance1-pro: Still creating... [30s elapsed]
aws_instance.instance1-pro: Creation complete after 34s [id=i-070b3ddec078f7cf6]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

$ terraform show
# aws_instance.instance1-pro:
resource "aws_instance" "instance1-pro" {
    ami                          = "ami-0ed28656d62ce20d6"
[...]
    primary_network_interface_id = "eni-0910bda45f1ab6f28"
    private_dns                  = "ip-172-31-57-175.ec2.internal"
    private_ip                   = "172.31.57.175"
    public_dns                   = "ec2-100-26-136-89.compute-1.amazonaws.com"
    public_ip                    = "100.26.136.89"
[...]

$ ssh ubuntu@100.26.136.89

ubuntu@ip-172-31-57-175:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
cc-eal        yes       n/a       Common Criteria EAL2 Provisioning Packages
cis-audit     no        —         Center for Internet Security Audit Tools
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance
fips          yes       n/a       NIST-certified FIPS modules
fips-updates  yes       n/a       Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <redacted>
           Subscription: <redacted>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

The process is slightly different in Microsoft Azure. You have to specify the marketplace plan information every time you want to use a marketplace product. This is done by adding 3 pieces of information to your existing Terraform code. Here's an example of launching an Ubuntu Pro 18.04 instance in Azure with Terraform. You can get more details in my article [Create an Ubuntu Pro VM with infrastructure in Azure using Terraform](https://davecore82.github.io/Create-an-Ubuntu-Pro-VM-with-infrastructure-in-Azure-using-Terraform/) but the TL;DR is that you just need to add the plan info to your Terraform configuration like this and specify the Ubuntu Pro sku instead of the Ubuntu LTS sku:

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

Ubuntu Pro is now available on Google Cloud too but I didn't get a chance to launch a Pro instance programatically yet. I'll report back when I do.

