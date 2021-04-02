---
layout: post
title: How to use the CIS Compliance Tools from Canonical on Ubuntu Pro 18.04
---

![](https://miro.medium.com/max/1271/1*caKlXtEF1f5MXn9iw0JfFg.jpeg)

Ubuntu Pro on  [AWS](https://ubuntu.com/aws/pro)  and  [Azure](https://ubuntu.com/azure/pro)  (and now on  [Google Cloud](https://ubuntu.com/gcp/pro)  too!) comes with the Ubuntu Advantage client that allows users to enable features of  [Ubuntu Advantage](https://ubuntu.com/advantage).

Here’s a quick guide on how to use the CIS beta feature to enable the CIS compliance tools from Canonical on Ubuntu Pro 18.04.

The first step is to launch an Ubuntu Pro 18.04 instance. I tested this procedure with both Azure and AWS. You can find the different Ubuntu Pro releases in the marketplace of each cloud.

After launching an Ubuntu Pro 18.04 instance, check the version of the UA client:

```
ubuntu@bionicpro:~$ ua version  
26.2~18.04.1
```

You want to be running at least version 26.2. If you have a lower version, you’ll want to add the  [UA Client Stable PPA from Launchpad](https://launchpad.net/~ua-client/+archive/ubuntu/stable)  and upgrade your UA packages:

```
ubuntu@ip-172-31-53-238:~$ sudo add-apt-repository ppa:ua-client/stableubuntu@ip-172-31-53-238:~$ sudo apt install ubuntu-advantage-tools ubuntu-advantage-proubuntu@ip-172-31-53-238:~$ ua version  
26.2~18.04.1
```

Once you have the UA client version 26.2 enabled, you can run ua status with the  `--all`  flag and see the cis feature:

```
ubuntu@bionicpro:~$ ua status --all  
SERVICE       ENTITLED  STATUS    DESCRIPTION  
cc-eal        yes       n/a       Common Criteria EAL2 Provisioning Packages  
cis           yes       disabled  Center for Internet Security Audit Tools  
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)  
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)  
fips          yes       disabled  NIST-certified FIPS modules  
fips-updates  yes       disabled  Uncertified security updates to FIPS modules  
livepatch     yes       enabled   Canonical Livepatch service
```

Note: It’s possible you might not see cis. On AWS, where I had to update the UA client, I couldn’t see cis, but the next step worked anyway.

Now you’re ready to enable CIS with the`--beta`flag:

```
ubuntu@bionicpro:~$ sudo ua enable cis --beta  
One moment, checking your subscription first  
Updating package lists  
Installing CIS Audit packages  
CIS Audit enabled
```

You can confirm that the usg packages have been installed:

```
ubuntu@bionicpro:~$ dpkg -l | grep usg  
ii  usg-cisbenchmark                       18.04.12                                    all          SCAP content for CIS Ubuntu Benchmarks  
ii  usg-common                             18.04.12                                    all          The CPE files for Ubuntu SCAP Content
```

And now you can proceed with the next steps from the “Configure and run CIS Benchmark rule” section of the Ubuntu’s documentation about  [CIS for Ubuntu 18.04 and Ubuntu 16.04](https://security-certs.docs.ubuntu.com/en/cis-18-16).
