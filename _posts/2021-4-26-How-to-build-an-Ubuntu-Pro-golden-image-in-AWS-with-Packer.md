---
layout: post
title: How to build an Ubuntu Pro golden image in AWS with Packer
---

This article explains how to build an Ubuntu Pro golden image in AWS with Packer. We will use Ubuntu Pro 20.04 as the base for the image, but this process can be done on any version of Ubuntu Pro. 

To be able to use Ubuntu Pro as a base in AWS, all you need to do is subscribe to the marketplace product before-hand. You can accept the terms and subscribe by finding the product in the AWS Marketplace and subscribing to it. For example, in the AWS services list, select AWS Marketplace Subscriptions. Click on Discover products to the left and search for ubuntu pro. Pick the version you need (ie. Ubuntu Pro 20.04 LTS) and on the page of that product click on Continue to Subscribe. In the Subscribe to this software page, click on Accept Terms to accept the extra pricing and terms and wait a few minutes for the subscription to complete.

Once you have the subscription active, you can proceed to building your base image with Packer. The example I use is taken from the [Build an Image page](https://learn.hashicorp.com/tutorials/packer/getting-started-build-image) on the Hashicorp website. 

Create a Packer template called `example-pro.pkr.hcl` with the following content:

```
variable "ami_name" {
  type    = string
  default = "my-custom-pro-ami"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }

source "amazon-ebs" "example" {
  ami_name      = "packer example ${local.timestamp}"
  instance_type = "t2.micro"
  region        = "us-east-1"
  source_ami = "ami-0ed28656d62ce20d6"
  ssh_username = "ubuntu"
}

build {
  sources = ["source.amazon-ebs.example"]

  provisioner "file" {
    destination = "/home/ubuntu/"
    source      = "./welcome.txt"
  }
  provisioner "shell" {
    inline = ["sudo ua detach --assume-yes"]
  }
}
```

The source AMI `ami-0ed28656d62ce20d6` in the above example corresponds to an Ubuntu Pro 20.04 AMI created by Canonical on 2020-10-22 as we can see with the following command:

```console
$ aws ec2 describe-images --filters "Name=name,Values=ubuntu-pro-server/images/*20.04*" | grep -i -e imageid -e creationdate -e \"name\"

            "CreationDate": "2020-08-14T09:20:41.000Z",
            "ImageId": "ami-01e2764da55c94742",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-focal-20.04-amd64-pro-serve-ae7ed378-8838-4fcf-842d-d1d09b34f116-ami-01575b6f5a4085419.4",

            "CreationDate": "2020-10-22T19:42:54.000Z",
            "ImageId": "ami-0ed28656d62ce20d6",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-focal-20.04-amd64-pro-serve-ae7ed378-8838-4fcf-842d-d1d09b34f116-ami-0d173ef9b96def311.4",

            "CreationDate": "2020-09-10T21:28:42.000Z",
            "ImageId": "ami-0fb29f6090e62a703",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-focal-20.04-amd64-pro-serve-ae7ed378-8838-4fcf-842d-d1d09b34f116-ami-08503d07277c98720.4",

```

In the `build {}` section of the Packer template above, we can see we use the `file` provisioner to copy a welcome.txt file inside the instance. This can be anything customization you want. 

The one post-command that is recommended to run to create a golden image in the cloud based on Ubuntu Pro is `ua detach`. This will make sure that the golden image base instance doesn't propagate its token information to all your other instances. 

Once your Packer template is created, you can build your golden image with `packer build`:

```console
$ packer build example-pro.pkr.hcl
amazon-ebs.example: output will be in this color.

==> amazon-ebs.example: Prevalidating any provided VPC information
==> amazon-ebs.example: Prevalidating AMI Name: packer example 20210426194519
    amazon-ebs.example: Found Image ID: ami-0ed28656d62ce20d6
==> amazon-ebs.example: Creating temporary keypair: packer_6087184f-3704-3a74-a24d-9aea48dc6645
==> amazon-ebs.example: Creating temporary security group for this instance: packer_60871851-056f-eacd-a0c5-7f88b5dd0865
==> amazon-ebs.example: Authorizing access to port 22 from [0.0.0.0/0] in the temporary security groups...
==> amazon-ebs.example: Launching a source AWS instance...
==> amazon-ebs.example: Adding tags to source instance
    amazon-ebs.example: Adding tag: "Name": "Packer Builder"
    amazon-ebs.example: Instance ID: i-0fe280e360cbd4da3
==> amazon-ebs.example: Waiting for instance (i-0fe280e360cbd4da3) to become ready...
==> amazon-ebs.example: Using ssh communicator to connect: 3.86.39.143
==> amazon-ebs.example: Waiting for SSH to become available...
==> amazon-ebs.example: Connected to SSH!
==> amazon-ebs.example: Uploading ./welcome.txt => /home/ubuntu/
    amazon-ebs.example: welcome.txt 53 B / 53 B [=============================================================================================================================================] 100.00% 0s
==> amazon-ebs.example: Provisioning with shell script: /tmp/packer-shell243199400
    amazon-ebs.example: This machine is now detached
==> amazon-ebs.example: Stopping the source instance...
    amazon-ebs.example: Stopping instance
==> amazon-ebs.example: Waiting for the instance to stop...
==> amazon-ebs.example: Creating AMI packer example 20210426194519 from instance i-0fe280e360cbd4da3
    amazon-ebs.example: AMI: ami-048dd96f85f54811b
==> amazon-ebs.example: Waiting for AMI to become ready...
==> amazon-ebs.example: Terminating the source AWS instance...
==> amazon-ebs.example: Cleaning up any extra volumes...
==> amazon-ebs.example: No volumes to clean up, skipping
==> amazon-ebs.example: Deleting temporary security group...
==> amazon-ebs.example: Deleting temporary keypair...
Build 'amazon-ebs.example' finished after 3 minutes 7 seconds.

==> Wait completed after 3 minutes 7 seconds

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs.example: AMIs were created:
us-east-1: ami-048dd96f85f54811b
```

And that's it. You can see Packer created a custom AMI ami-048dd96f85f54811b in region us-east-1. After you create a new instance from that AMI, your new instance will be automatically attached to Ubuntu Advantage as you can see here:

```console
$ ua status
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

