---
layout: post
title: Common Criteria on Ubuntu Pro 16.04
---

I've been trying without success to enable and use Common Criteria on Ubuntu Pro 16.04. Trying to run the beta command `sudo ua enable cc-eal --beta` errors out with the following error:

```console
ubuntu@xenialpro:~$ sudo ua enable cc-eal --beta
One moment, checking your subscription first
GPG key '/usr/share/keyrings/ubuntu-cc-keyring.gpg' not found
This is with the UA client version 26.2~16.04.1.
```

I filed a [bug under the UA client](https://github.com/canonical/ubuntu-advantage-client/issues/1527) project on Github for this issue. 

But I believe the problem comes from the fact that the Canonical tools for common criteria require precisely Ubuntu 16.04.4, so I don't think this would work anyway considering that we're already at version 16.04.7 in the latest Ubuntu Pro images.

As the [common criteria Ubuntu security documentation](https://security-certs.docs.ubuntu.com/en/cc-16) reports:

> Common criteria evaluated configuration is currently available for
> Ubuntu 16.04.4 LTS (Server).

This is particularly an issue for Ubuntu Pro customers who would want to use common criteria, since the latest Ubuntu Pro images already come with Ubuntu 16.04.7 and won't allow customers to enable common criteria even with the manual method described in the link above. As far as I know it's not trivial either to downgrade from Ubuntu 16.04.7 to 16.04.4.

Here is what happens on an Ubuntu Pro 16.04.7 system when trying to manually enable CC-EAL (without using the UA client):

```console
ubuntu@xenialpro:~$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys A166877412DAC26E73CEBF3FF6C280178D13028C
Executing: /tmp/tmp.naEcO4QzJW/gpg.1.sh --keyserver
keyserver.ubuntu.com
--recv-keys
A166877412DAC26E73CEBF3FF6C280178D13028C
gpg: requesting key 8D13028C from hkp server keyserver.ubuntu.com
gpg: key 8D13028C: "Launchpad PPA for ubuntu-advantage" not changed
gpg: Total number processed: 1
gpg: unchanged: 1

ubuntu@xenialpro:~$ sudo add-apt-repository -u 'deb https://<user>:<password>@private-ppa.launchpad.net/ubuntu-advantage/commoncriteria/ubuntu xenial main'

ubuntu@xenialpro:~$ sudo apt install ubuntu-commoncriteria
[...]
Unpacking ubuntu-commoncriteria (1.0.16.04.1) ...
Setting up ubuntu-commoncriteria (1.0.16.04.1) ...

ubuntu@xenialpro:~$ mkdir cc-dir
ubuntu@xenialpro:~$ cd cc-dir

ubuntu@xenialpro:~/cc-dir$ cp /usr/lib/common-criteria/Configure-Ubuntu-16.04-Common-Criteria.sh ./
ubuntu@xenialpro:~/cc-dir$ cp /usr/lib/common-criteria/Ubuntu-16.04-Common-Criteria.tar.gz ./

ubuntu@xenialpro:~/cc-dir$ sudo ./Configure-Ubuntu-16.04-Common-Criteria.sh Ubuntu-16.04-Common-Criteria.tar.gz
Log file: /var/log/CC-EAL2-Ubuntu-16.04.4_20210406201104.log

This script will configure the system to ensure it's compliant
with Common Criteria EAL2.

Do you want to proceed? [N/y] y
Checking system...
This script should be run in a Ubuntu 16.04.4 system
```

And the script exits with an error code 1. 

I filed a [bug on Launchpad](https://bugs.launchpad.net/ubuntu-security-certifications/+bug/1922796) for this more general common criteria issue.

I was only able to make common criteria work with the manual method on an Ubuntu 16.04.4 machine, whether by launching an old Ubuntu Pro image (like Canonical:UbuntuServer:16.04-LTS:16.04.201804180 on Azure) or by launching a VM in KVM/VirtualBox with the Ubuntu 16.04.4 ISO. 


**UPDATE 2021-04-08:**  I continued to find a way to make common criteria work on Ubuntu Pro. My first idea was to find an older Ubuntu Pro 16.04 image that would have been built around April 2018 (about the time 16.04.4 came out) but that's not going to happen because Ubuntu Pro itself was announced in December 2019.

So my second idea was to eliminate the check for the 16.04.4 exact dot release in the script. This did work as far as getting the script to complete, but I'm sure this is untested and would be unsupported by Canonical.

In any case, I edited the script `Configure-Ubuntu-16.04-Common-Criteria.sh` and commented out the following line:

```bash
#if [[ "$DISTRIB_DESCRIPTION" != 'Ubuntu 16.04.4 LTS' ]]; then
#       abort "This script should be run in a Ubuntu 16.04.4 system"
#fi
```

I then ran the script again:

```console
$ sudo ./Configure-Ubuntu-16.04-Common-Criteria.sh Ubuntu-16.04-Common-Criteria.tar.gz
```

But it ended with this error after just a few seconds:

```
Some of the required packages are not installed: ebtables libvirt-bin qemu-kvm
```

So I installed those packages and ran the script again:

```console
$ sudo apt install ebtables libvirt-bin qemu-kvm
$ sudo ./Configure-Ubuntu-16.04-Common-Criteria.sh Ubuntu-16.04-Common-Criteria.tar.gz
```
And this time it went through all the way:
```
Log file: /var/log/CC-EAL2-Ubuntu-16.04.4_20210408153319.log

This script will configure the system to ensure it's compliant
with Common Criteria EAL2.

Do you want to proceed? [N/y] y
Checking system...
Decompressing tarball...
Checking tarball contents...
Installing PPA key...
Adding temporary APT repository...
Running apt-get update... 
Checking for installed packages...
Installing additional packages...
Removing non-compliant packages...
Removing "unattended-upgrades"...
Removing "apport-symptoms"...
Running post installation scripts...
Running post-install script, setumask...
Running post-install script, config-fstab...
Running post-install script, config-auditd...
Running post-install script, config-bootloader...
Running post-install script, config-sshd...
Running post-install script, config-modprobe...
Running post-install script, config-libvirt...
Running post-install script, config-qemu...
Running post-install script, config-apparmor...
Running post-install script, config-pam...
Running post-install script, screen...
Running post-install script, permissions...
Running post-install script, config-alias...
Running post-install script, config-hold-packages...
Common Criteria EAL2 configuration has successfully completed.
The system must reboot for the configuration to take effect.

Reboot the system now? [N/y] y
Rebooting...
```

After the reboot I noticed this message when I SSH'd back in:
```
Starting session in 10 seconds
```
And there's a log file with the results:
```console
$ head CC-EAL2-Ubuntu-16.04.4_20210408153319.log
This script will configure the system to ensure it's compliant
with Common Criteria EAL2.
Checking system...
Decompressing tarball...
Checking tarball contents...
Installing PPA key...
Adding temporary APT repository...
Running apt-get update... 
Get:1 file:/tmp/cc.Gj8E5U1bjX/mirror xenial InRelease [24.3 kB]
Hit:2 http://azure.archive.ubuntu.com/ubuntu xenial InRelease

$ tail CC-EAL2-Ubuntu-16.04.4_20210408153319.log
/tmp/cc.Gj8E5U1bjX/post-inst/config-hold-packages: zlib1g set on hold.
/tmp/cc.Gj8E5U1bjX/post-inst/checkerror: --- Starting execution of /tmp/cc.Gj8E5U1bjX/post-inst/checkerror ---
/tmp/cc.Gj8E5U1bjX/post-inst/checkerror: Common Criteria Evaluated Configuration
Common Criteria Evaluated Configuration
/tmp/cc.Gj8E5U1bjX/post-inst/checkerror: successfully established
successfully established
Common Criteria EAL2 configuration has successfully completed.
The system must reboot for the configuration to take effect.
Rebooting...
./Configure-Ubuntu-16.04-Common-Criteria.sh: line 274: 13348 Terminated              reboot
```

Just to reiterate, this method is not tested/recommended/supported by Canonical. 

