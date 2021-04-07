---
layout: post
title: How to use the DISA STIG tools from Canonical on Ubuntu Pro 16.04
---

Following up on my article [How to use the CIS Compliance Tools from Canonical on Ubuntu Pro 18.04](https://davecore82.github.io/How-to-use-the-CIS-Compliance-Tools-from-Canonical-on-Ubuntu-Pro-18.04/), here's a quick guide on how to get started with STIG on Ubuntu Pro 16.04.

Security Technical Implementation Guides (STIG) are developed by the Defense Information System Agency (DISA) for the U.S. Department of Defense (DoD). They are configuration guidelines for hardening systems to improve security. They contain technical guidance which when implemented, locks down software and systems to mitigate malicious attacks.

As of this writing, DISA has, in conjunction with Canonical, developed STIGs for Ubuntu 16.04 LTS and Ubuntu 18.04 LTS. However Canonical currently only has audit tooling for STIG on Ubuntu 16.04.

Ubuntu Pro on  [AWS](https://ubuntu.com/aws/pro)  and  [Azure](https://ubuntu.com/azure/pro)  (and now on  [Google Cloud](https://ubuntu.com/gcp/pro)  too!) comes with the Ubuntu Advantage client that allows users to enable features of  [Ubuntu Advantage](https://ubuntu.com/advantage).

The first step is to launch an Ubuntu Pro 16.04 instance. I tested this procedure with Azure. You can find the different Ubuntu Pro releases in the marketplace of each cloud.

After launching an Ubuntu Pro 16.04 instance, check the version of the UA client:

```console
$ ua version  
26.2~16.04.1
```

I ran these tests on version 26.2. If you have a lower version, you might need to add the  [UA Client Stable PPA from Launchpad](https://launchpad.net/~ua-client/+archive/ubuntu/stable)  and upgrade your UA packages:

```console
$ sudo add-apt-repository ppa:ua-client/stable
$ sudo apt install ubuntu-advantage-tools ubuntu-advantage-pro

$ ua version  
26.2~16.04.1
```

Once you have the UA client version 26.2 enabled, you can run ua status with the `--all` flag and see the cis feature:

```console
$ ua status --all

SERVICE       ENTITLED  STATUS    DESCRIPTION
cc-eal        yes       disabled  Common Criteria EAL2 Provisioning Packages
cis           yes       disabled  Center for Internet Security Audit Tools
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)
fips          yes       n/a       NIST-certified FIPS modules
fips-updates  yes       n/a       Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service
```

For STIG, we need the usg-stig package, which is in the same repository as the CIS packages. So we'll need to enable CIS with the `--beta` flag (this only installs the repositories needed for the CIS tooling, it doesn't actually apply any kind of CIS configuration to the system):

```console
$ sudo ua enable cis --beta  

One moment, checking your subscription first  
Updating package lists  
Installing CIS Audit packages  
CIS Audit enabled
```

You can confirm that the usg packages have been installed:

```console
$ dpkg -l | grep usg

ii  usg-cisbenchmark                    16.04.7                                       all          SCAP content for CIS Ubuntu Benchmarks
ii  usg-common                          16.04.7                                       all          The CPE files for Ubuntu SCAP Content
```

Now install the usg-stig package:

```console
$ sudo apt install usg-stig

Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  grub-pc-bin
Use 'sudo apt autoremove' to remove it.
The following NEW packages will be installed:
  usg-stig
0 upgraded, 1 newly installed, 0 to remove and 7 not upgraded.
Need to get 80.4 kB of archives.
After this operation, 1,322 kB of additional disk space will be used.
Get:1 https://esm.ubuntu.com/cis/ubuntu xenial/main amd64 usg-stig all 16.04.7 [80.4 kB]
Fetched 80.4 kB in 0s (372 kB/s)
Selecting previously unselected package usg-stig.
(Reading database ... 54785 files and directories currently installed.)
Preparing to unpack .../usg-stig_16.04.7_all.deb ...
Unpacking usg-stig (16.04.7) ...
Setting up usg-stig (16.04.7) ...
```

There's a README file that comes in the usg-common package that explains more about the STIG example (you can view it with `zcat /usr/share/doc/usg-common/README.audit.gz`). Here are the important steps:

```console
$ cd /usr/share/ubuntu-scap-security-guides

$ sudo oscap xccdf eval --profile \
>      xccdf_com.ubuntu.xenial.stig_profile_MAC-1_Classified \
>      --cpe Ubuntu_16.04_Benchmark-cpe-dictionary.xml \
>      --results stig-results.xml \
>      U_Canonical_Ubuntu_16-04_LTS_STIG_V1R1_Manual-xccdf.xml
```

And you can even generate an HTML report that looks like this:
![DISA STIG HTML report](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/disa-stig-html-report.png)

```console
$ sudo oscap xccdf generate report stig-results.xml | sudo tee stig-report.html
```


