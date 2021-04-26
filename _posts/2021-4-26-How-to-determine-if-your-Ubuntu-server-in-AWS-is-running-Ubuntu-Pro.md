---
layout: post
title: How to determine if your Ubuntu server in AWS is running Ubuntu Pro
---

This article explains how to determine if your Ubuntu server in AWS is running Ubuntu Pro.

This article is based on a [question from Stack Overflow](https://stackoverflow.com/questions/62629568/how-to-determine-if-your-server-is-running-ubuntu-pro).  The question stated:

> I've taken over administering an EC2 instance created by someone who's left the company. I understand that the server may be operating Ubuntu Pro but am having trouble working out how to check.

A user called stefansundin gave this answer:

> Since this is a marketplace product, you can go to AWS Marketplace Subscriptions to see what marketplace products are currently running. You should also be able to tell from the AMI. If you look at the instance details in the EC2 console, there is an AMI field. If you click the AMI name, it may say if it is Ubuntu Pro or not. You may also be able to tell by looking at your AWS Bill. This marketplace product appears to have an extra cost (albeit small). It may not tell you which instance though, in case you are running more than one in the account.

I posted another answer and wanted to include it on my blog here:

You can find the the marketplace product code that your instance is using with the following methods:

With Instance Metadata Service (IMDS) version 1:

`curl http://169.254.169.254/latest/dynamic/instance-identity/document`

With IMDSv2:

```
TOKEN=`curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"` \
&& curl -H "X-aws-ec2-metadata-token: $TOKEN" -v http://169.254.169.254/latest/dynamic/instance-identity/document
```

This also seems to work:

`curl http://169.254.169.254/latest/meta-data/product-codes`

This will return a product code like `abwe6hiebq70xss7fo2neefpy` and you can then go to `https://aws.amazon.com/marketplace/pp?sku=<product code here>` which will return an AWS marketplace page like [Ubuntu Pro 20.04 LTS](https://aws.amazon.com/marketplace/pp?sku=abwe6hiebq70xss7fo2neefpy).

