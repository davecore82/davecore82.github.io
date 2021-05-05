---
layout: post
title: CIS scoring comparison of the Ubuntu 16.04 AMI from CIS compared to the Ubuntu Pro 16.04 AMI on the AWS Marketplace
---

This article does a quick level 1 CIS scoring comparison of the [CIS Ubuntu Linux 16.04 LTS Benchmark - Level 1](https://aws.amazon.com/marketplace/pp/B078TPPXV2?ref=cns_srchrow) AMI from CIS on the AWS Marketplace compared to the [Ubuntu Pro 16.04 LTS](https://aws.amazon.com/marketplace/pp/B0821WW873?ref=cns_srchrow) AMI on the AWS Marketplace with the included CIS hardening script ran on top of it. 

The CIS auditing tooling used to score both systems comes from Canonical. Instructions for the auditing can be found in [Canonical's documentation](https://security-certs.docs.ubuntu.com/en/cis-audit). The command that was ran on both systems was `cis-audit level1_server`. This test was done on May 5th 2021 with the latest version of each AMI.

## HTML results
Here is the result for the **CIS Ubuntu Linux 16.04 LTS Benchmark - Level 1** AMI:

![cis-16.04-report-ImageFromCIS HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis-16.04-report-ImageFromCIS.png)

The 9 tests that failed are:

 - Ensure permissions on bootloader config are configured		 
 - Ensure bootloader password is set
 - Ensure authentication required for single user mode
 - Ensure /etc/hosts.allow is configured
 - Ensure rsyslog is configured to send logs to a remote log host
 - Ensure permissions on all logfiles are configured
 - Ensure lockout for failed password attempts is configured
 - Ensure password reuse is limited
 - Ensure access to the su command is restricted

Here is the result for the **Ubuntu Pro 16.04 LTS AMI with the included CIS hardening script** ran on top of it:

![cis-16.04-report-UbuntuProAfterHardening HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis-16.04-report-UbuntuProAfterHardening.png)

The 6 tests that failed are:

 - Ensure bootloader password is set
 - Ensure authentication required for single user mode
 - Ensure default deny firewall policy
 - Ensure firewall rules exist for all open ports
 - Ensure permissions on all logfiles are configured
 - Ensure SSH access is limited

And just for reference, here is the same Ubuntu Pro 16.04 LTS AMI but **without the included CIS hardening script**. I ran this script right after launching an Ubuntu Pro 16.04 LTS instance:

![cis-16.04-report-UbuntuProBeforeHardening HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis-16.04-report-UbuntuProBeforeHardening.png)

## Conclusion
We can see that the CIS hardening script included with Ubuntu Pro can bring a system to a CIS level 1 score that's even higher than the hardened image from CIS. Users could take further action and configure extra rules in the ruleset.params file to fix some of those failed tests (ie. Boot loader passwd, single user login, ssh access is limited, etc.). 

If CIS score is important to a user, it's important to take the time to apply this CIS hardening to your cloud image. Please refer to my previous articles [How to build an Ubuntu Pro golden image in AWS with Packer](https://davecore82.github.io/How-to-build-an-Ubuntu-Pro-golden-image-in-AWS-with-Packer/) and [How to use the CIS Compliance Tools from Canonical on Ubuntu Pro 18.04](https://davecore82.github.io/How-to-use-the-CIS-Compliance-Tools-from-Canonical-on-Ubuntu-Pro-18.04/) to learn how to create your own hardened Ubuntu Pro image in AWS with Packer.

