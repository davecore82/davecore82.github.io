---
layout: post
title: How to get ESM in Auto Scaling groups in AWS with Ubuntu Pro 16.04
---

Ubuntu 16.04 LTS will enter the  [extended security maintenance (ESM)](https://ubuntu.com/security/esm?utm_source=blog)  period in April 2021. Auto Scaling groups in AWS contain a collection of Amazon EC2 instances that are treated as a logical grouping for the purposes of automatic scaling and management. Ubuntu Pro is a premium image designed by Canonical to provide additional coverage, like ESM, for production environments running in the cloud. This article explains how to use Ubuntu Pro 16.04 with auto scaling groups in AWS to continue to receive security updates after Ubuntu 16.04 LTS enters ESM.

The use case for this article is the following scenario: You currently have an application running in AWS that scales horizontally with auto scaling groups. You are currently using the AMI for Ubuntu 16.04 LTS but you heard that you will no longer receive security updates starting on May 1st 2021. Canonical provides security updates for an extra 3 years for Ubuntu 16.04 via its ESM program. ESM is available out of the box on Ubuntu Pro, so the goal here is to use Ubuntu Pro 16.04 instead of Ubuntu 16.04 LTS to continue to receive these security updates. On top of that you get other benefits like Kernel Livepatch which delivers kernel security patches to your running kernel without the need for reboot. 

This article assumes you already have an AWS acount and are familiar with the AWS console and auto scaling groups in general. Having access to the aws CLI tool will help too.

To create a Launch Configuration, click on “Services” on the navigation bar at the top of the screen and click on “EC2” to go to EC2 console. On the navigation pane on the left of the screen, under “Auto Scaling”, choose “Launch Configurations”.

On the next page, choose “Create launch configuration”. 

You'll need the AMI ID of Ubuntu Pro 16.04. You can get that with the following command:

```console
$ aws ec2 describe-images --filters "Name=name,Values=ubuntu-pro-server/images/*16.04*" | grep -i -e imageid -e creationdate -e \"name\"

            "CreationDate": "2020-09-10T21:30:52.000Z",
            "ImageId": "ami-045fb7fbf6664942e",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-xenial-16.04-amd64-pro-serv-cf418943-fe22-4b92-92b0-f4884b480f10-ami-0f44f2038867e7dd8.4",

            "CreationDate": "2020-08-14T09:22:28.000Z",
            "ImageId": "ami-05c01c1d3d4a7fd6a",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-xenial-16.04-amd64-pro-serv-cf418943-fe22-4b92-92b0-f4884b480f10-ami-00f454613d5c220e7.4",

            "CreationDate": "2021-02-09T14:16:05.000Z",
            "ImageId": "ami-07b11b5662a70ed2f",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-xenial-16.04-amd64-pro-serv-cf418943-fe22-4b92-92b0-f4884b480f10-ami-00a509450d3c29014.4",

            "CreationDate": "2020-10-22T19:43:45.000Z",
            "ImageId": "ami-09a128c4ab63114c1",
            "Name": "ubuntu-pro-server/images/hvm-ssd/ubuntu-xenial-16.04-amd64-pro-serv-cf418943-fe22-4b92-92b0-f4884b480f10-ami-0ce49229531215799.4",
```
For this example we'll take the latest one, so ami-07b11b5662a70ed2f.

On the Create launch configuration page, specify a name for your Launch configuration and enter ami-07b11b5662a70ed2f for the AMI. Choose an appropriate Instance Type (t2.small in my case) and specify other configurations you might need. In my case I'll leave the defaults but specify a key pair to SSH into the instance later. Click on Create launch configuration when you're ready.

Back on the Launch configurations page, select your newly created launch configuration and click on Actions and on Create Auto Scaling group.

On the Choose launch template or configuration page, enter a name for your Auto Scaling group and click Next. Choose your VPC and Subnets (I chose my default VPC and us-east-1a) and click Next. Configure the rest of the options as needed. For this example I won't use a load balancer or health checks. On the Configure group size and scaling policies page, choose your capacity settings. I will use a desired and minimum capacity of 2 and a Maximum capacity of 5. I use Target tracking scaling policy and will leave the defaults of Average CPU Utilization for the metric type and a target value of 50. Click Next. Choose what you need for the remaining pages and click on Create Auto Scaling group. 

You can then view the Auto Scaling group and created instances from the AWS console, but I'll switch to the aws CLI for the next steps. 

View your newly created Auto Scaling group:

```console
$ aws autoscaling describe-auto-scaling-groups --output table
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                                                                          DescribeAutoScalingGroups                                                                         |
+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
||                                                                             AutoScalingGroups                                                                            ||
|+----------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+|
||  AutoScalingGroupARN             |  arn:aws:autoscaling:us-east-1:870742490987:autoScalingGroup:32d8ee59-7fc5-4605-a466-2ddae676a921:autoScalingGroupName/xenialproasg   ||
||  AutoScalingGroupName            |  xenialproasg                                                                                                                         ||
||  CreatedTime                     |  2021-04-22T15:32:10.384000+00:00                                                                                                     ||
||  DefaultCooldown                 |  300                                                                                                                                  ||
||  DesiredCapacity                 |  2                                                                                                                                    ||
||  HealthCheckGracePeriod          |  300                                                                                                                                  ||
||  HealthCheckType                 |  EC2                                                                                                                                  ||
||  LaunchConfigurationName         |  xenialprolaunchconfig                                                                                                                ||
||  MaxSize                         |  5                                                                                                                                    ||
||  MinSize                         |  2                                                                                                                                    ||
||  NewInstancesProtectedFromScaleIn|  False                                                                                                                                ||
||  ServiceLinkedRoleARN            |  arn:aws:iam::870742490987:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling                               ||
||  VPCZoneIdentifier               |  subnet-5a16181c                                                                                                                      ||
|+----------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+|
|||                                                                            AvailabilityZones                                                                           |||
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
|||  us-east-1a                                                                                                                                                            |||
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
|||                                                                                Instances                                                                               |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||   AvailabilityZone   |  HealthStatus    |       InstanceId         |  InstanceType    |    LaunchConfigurationName    |  LifecycleState    |   ProtectedFromScaleIn    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||  us-east-1a          |  Healthy         |  i-056abf750d5371f69     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
|||  us-east-1a          |  Healthy         |  i-0f239108554dfc486     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||                                                                           TerminationPolicies                                                                          |||
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
|||  Default                                                                                                                                                               |||
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
```

I specified a desired and minimum capacity of 2 instances, so that's what I get after creating the ASG. We will now connect to each instance and look at the ESM feature provided by Ubuntu Pro.

Add port 22 to the inbound rules of your instances security group and SSH to your instance:

```console
$ ssh ubuntu@3.84.144.222
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-1119-aws x86_64)

UA Infra: Extended Security Maintenance (ESM) is enabled.

ubuntu@ip-172-31-0-145:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
cc-eal        yes       disabled  Common Criteria EAL2 Provisioning Packages
cis-audit     no        —         Center for Internet Security Audit Tools
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance
fips          yes       disabled  NIST-certified FIPS modules
fips-updates  yes       disabled  Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <redacted>
           Subscription: <redacted>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

We can see in the above output that esm-infra is enabled on my instance. This service is the one that will add 3 extra years of security updates to my Ubuntu 16.04 Pro instance. We see we also have esm-apps and livepatch enabled on top of that.

Running apt update shows me where these extra security updates come from:

```console
ubuntu@ip-172-31-0-145:~$ sudo apt update
[...]
Hit:7 https://esm.ubuntu.com/infra/ubuntu xenial-infra-security InRelease
Hit:8 https://esm.ubuntu.com/infra/ubuntu xenial-infra-updates InRelease
[...]
```

To simulate CPU activity, I'll install the stress utility on my 2 instances and run a stress test for 400 seconds:

```console
ubuntu@ip-172-31-0-145:~$ sudo apt-get -y install stress

ubuntu@ip-172-31-0-145:~$ stress --cpu 60 --timeout 400
stress: info: [4537] dispatching hogs: 60 cpu, 0 io, 0 vm, 0 hdd
```

From your machine when your aws commands (or in the AWS console), wait a few minutes and look at your Auto Scaling group again:

```console
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
|||                                                                                Instances                                                                               |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||   AvailabilityZone   |  HealthStatus    |       InstanceId         |  InstanceType    |    LaunchConfigurationName    |  LifecycleState    |   ProtectedFromScaleIn    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||  us-east-1a          |  Healthy         |  i-03ab3fd87c7425e89     |  t2.small        |  xenialprolaunchconfig        |  Pending           |  False                    |||
|||  us-east-1a          |  Healthy         |  i-056abf750d5371f69     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
|||  us-east-1a          |  Healthy         |  i-06ab549612ed289c3     |  t2.small        |  xenialprolaunchconfig        |  Pending           |  False                    |||
|||  us-east-1a          |  Healthy         |  i-0f239108554dfc486     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
```

You can see that 2 new instances are being created to handle the load on your instances. 

SSH to one of these new instances and check the UA status:

```console
$ ssh ubuntu@ec2-34-230-13-3.compute-1.amazonaws.com

ubuntu@ip-172-31-2-217:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
cc-eal        yes       disabled  Common Criteria EAL2 Provisioning Packages
cis-audit     no        —         Center for Internet Security Audit Tools
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance
fips          yes       disabled  NIST-certified FIPS modules
fips-updates  yes       disabled  Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service

Enable services with: ua enable <service>

                Account: <redacted>
           Subscription: <redacted>
            Valid until: 9999-12-31 00:00:00
Technical support level: essential
```

We see that all new instances created with this Auto Scaling group will have Ubuntu Advantage services like esm-infra, esm-apps and livepatch automatically enabled.

After a few minutes with low CPU activity, the ASG will scale back down as needed:

```
||+------------------------------------------------------------------------------------------------------------------------------------------------------------------------+||
|||                                                                                Instances                                                                               |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||   AvailabilityZone   |  HealthStatus    |       InstanceId         |  InstanceType    |    LaunchConfigurationName    |  LifecycleState    |   ProtectedFromScaleIn    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
|||  us-east-1a          |  Healthy         |  i-03ab3fd87c7425e89     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
|||  us-east-1a          |  Healthy         |  i-056abf750d5371f69     |  t2.small        |  xenialprolaunchconfig        |  Terminating       |  False                    |||
|||  us-east-1a          |  Healthy         |  i-06ab549612ed289c3     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
|||  us-east-1a          |  Healthy         |  i-0f239108554dfc486     |  t2.small        |  xenialprolaunchconfig        |  InService         |  False                    |||
||+----------------------+------------------+--------------------------+------------------+-------------------------------+--------------------+---------------------------+||
```

Changing your AWS Auto Scaling Launch configurations to use Ubuntu Pro 16.04 instead of Ubuntu 16.04 LTS is great because it's a small change and it lets you continue to use your familiar Ubuntu 16.04 workflows for up to another 3 years, which should give you enough time to plan your upgrade to a more recent release of Ubuntu like the latest 20.04 LTS. 

Find out more information here:

 - [Ubuntu Pro for AWS](https://ubuntu.com/aws/pro)
 - [Extended Security Maintenance](https://ubuntu.com/security/esm) (ESM)
 - [AWS Auto Scaling](https://aws.amazon.com/autoscaling/)
 - [How to setup EC2 Auto Scaling Group (ASG) on AWS](https://www.howtoforge.com/how-to-setup-ec2-auto-scaling-group-asg-on-aws/) - 

