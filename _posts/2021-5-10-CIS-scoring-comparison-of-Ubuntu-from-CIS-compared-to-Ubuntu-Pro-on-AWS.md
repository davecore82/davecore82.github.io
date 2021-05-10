---
layout: post
title: CIS scoring comparison of the Ubuntu 16.04, 18.04 and 20.04 AMIs from CIS compared to Ubuntu Pro on AWS
---

This article gives a quick overview of level 1 CIS scoring comparison of the CIS Ubuntu images from CIS on the AWS Marketplace compared to Ubuntu Pro also on the AWS Marketplace with the included CIS hardening script ran on top of it. We will do a comparison of Ubuntu 16.04, 18.04 and 20.04.

This test was done on May 10th 2021 with the latest version of each AMI. For extra details on how the tests were ran, please check my previous article [CIS scoring comparison of the Ubuntu 16.04 AMI from CIS compared to the Ubuntu Pro 16.04 AMI on the AWS Marketplace](https://davecore82.github.io/CIS-scoring-comparison-of-the-Ubuntu-16.04-AMI-from-CIS-compared-to-the-Ubuntu-Pro-16.04-AMI-on-the-AWS-Marketplace/).

Note that Ubuntu Pro does not come with CIS hardening out of the box. The included CIS level 1 server hardening script needs to be ran to obtain the following results.

## TL;DR
Here are the results in table format:

![cis comparison table](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis-comparison-table.png)


## CIS Ubuntu Linux 20.04 LTS Benchmark - Level 1 AMI
![cis-20.04-report-ImageFromCIS HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis2004.png)

## Ubuntu Pro 20.04 AMI with hardening script
![cis-20.04-report-Pro HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/pro2004.png)

## CIS Ubuntu Linux 18.04 LTS Benchmark - Level 1 AMI
![cis-18.04-report-ImageFromCIS HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis1804.png)

## Ubuntu Pro 18.04 AMI with hardening script
![cis-18.04-report-Pro HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/pro1804.png)

## CIS Ubuntu Linux 16.04 LTS Benchmark - Level 1 AMI
![cis-16.04-report-ImageFromCIS HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/cis1604.png)

## Ubuntu Pro 16.04 AMI with hardening script
![cis-16.04-report-Pro HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/pro1604.png)


## Conclusion
We can see that the CIS hardening script included with Ubuntu Pro can bring a system to a CIS level 1 score that's even higher than the hardened image from CIS. Users could take further action and configure extra rules in the ruleset.params file to fix some of those failed tests (ie. Boot loader passwd, single user login, ssh access is limited, etc.). 

If CIS score is important to a user, it's important to take the time to apply CIS hardening to your cloud image. Please refer to my previous articles [How to build an Ubuntu Pro golden image in AWS with Packer](https://davecore82.github.io/How-to-build-an-Ubuntu-Pro-golden-image-in-AWS-with-Packer/) and [How to use the CIS Compliance Tools from Canonical on Ubuntu Pro 18.04](https://davecore82.github.io/How-to-use-the-CIS-Compliance-Tools-from-Canonical-on-Ubuntu-Pro-18.04/) to learn how to create your own hardened Ubuntu Pro image in AWS with Packer.

