---
layout: post
title: How to build an Ubuntu Pro golden image in GCP with Packer
---

This article explains how to build an Ubuntu Pro golden image in Google Cloud with Packer. We will use Ubuntu Pro 18.04 as the base for the image, but this process can be done on any version of Ubuntu Pro. 

Unlike with AWS where you have to subscribe to the marketplace product before-hand or in Azure where you have to specify plan information each time you launch a marketplace image, there is nothing you need to do in Google Cloud to use a marketplace image. 

So just go ahead and create a Packer template. I used the Packer JSON format. This is a very basic example. Create a file called `ubuntu-pro-gcp.json` with the following content:

```
{
  "builders": [
    {
      "type": "googlecompute",
      "project_id": "<your project ID that will be used to launch instances and store images>",
      "source_image_project_id": "ubuntu-os-pro-cloud",
      "source_image_family": "ubuntu-pro-1804-lts",
      "zone": "us-central1-a",
      "image_description": "created-with-packer",
      "ssh_username": "root",
      "tags": "packer",
      "account_file": "<your google credentials in json format>"
    }
  ],
  "provisioners": [
    {
      "type": "shell",
      "inline": [
        "sudo ua status --wait",
        "sudo ua detach --assume-yes"
      ]
    }
  ]
}

```

You can get the list of source image families for Ubuntu Pro in the [google documentation](https://cloud.google.com/compute/docs/images/os-details#ubuntu_pro). 

The one post build operation that is recommended to create a golden image in the cloud based on Ubuntu Pro is `ua detach`. This will make sure that the golden image base instance doesn't propagate its token information to all your other instances. In the above example we use `sudo ua status --wait` to wait for the original UA attach to finish (only available in the ubuntu-advantage-tools package version 26.2+) and then the `sudo ua detach --assume-yes` command to detach the subscription. 

Once your Packer template is created, you can build your golden image with `packer build`:

```console
$ packer build ubuntu-pro-gcp.json 
googlecompute: output will be in this color.

==> googlecompute: Checking image does not exist...
==> googlecompute: Creating temporary rsa SSH key for instance...
==> googlecompute: Using image: ubuntu-pro-1804-bionic-v20210423
==> googlecompute: Creating instance...
    googlecompute: Loading zone: us-central1-a
    googlecompute: Loading machine type: n1-standard-1
    googlecompute: Requesting instance creation...
    googlecompute: Waiting for creation operation to complete...
    googlecompute: Instance has been created!
==> googlecompute: Waiting for the instance to become running...
    googlecompute: IP: 34.69.108.47
==> googlecompute: Using ssh communicator to connect: 34.69.108.47
==> googlecompute: Waiting for SSH to become available...
==> googlecompute: Connected to SSH!
==> googlecompute: Provisioning with shell script: /tmp/packer-shell627635836
    googlecompute: .............
    googlecompute: SERVICE       ENTITLED  STATUS    DESCRIPTION
    googlecompute: esm-apps      yes                enabled            UA Apps: Extended Security Maintenance (ESM)
    googlecompute: esm-infra     yes                enabled            UA Infra: Extended Security Maintenance (ESM)
    googlecompute: fips          yes                n/a                NIST-certified FIPS modules
    googlecompute: fips-updates  yes                n/a                Uncertified security updates to FIPS modules
    googlecompute: livepatch     yes                enabled            Canonical Livepatch service
    googlecompute:
    googlecompute: Enable services with: ua enable <service>
    googlecompute:
    googlecompute:                 Account: <redacted>
    googlecompute:            Subscription: <redacted>
    googlecompute:             Valid until: 9999-12-31 00:00:00
    googlecompute: Technical support level: essential
    googlecompute: Detach will disable the following services:
    googlecompute:     esm-apps
    googlecompute:     esm-infra
    googlecompute:     livepatch
    googlecompute: Updating package lists
    googlecompute: Updating package lists
    googlecompute: This machine is now detached
==> googlecompute: Deleting instance...
    googlecompute: Instance has been deleted!
==> googlecompute: Creating image...
==> googlecompute: Deleting disk...
    googlecompute: Disk has been deleted!
Build 'googlecompute' finished after 3 minutes 42 seconds.

==> Wait completed after 3 minutes 42 seconds

==> Builds finished. The artifacts of successful builds are:
--> googlecompute: A disk image was created: packer-1619799495
```

And that's it. You can see Packer created a custom image packer-1619799495 in region us-central1-a. After you create a new instance from that image, your new instance will be automatically attached to Ubuntu Advantage as you can see here:

```console
ubuntu@davecore-pro-from-packer-with-ua-detach:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)
fips          yes       n/a       NIST-certified FIPS modules
fips-updates  yes       n/a       Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <redacted>
           Subscription: <redacted>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

