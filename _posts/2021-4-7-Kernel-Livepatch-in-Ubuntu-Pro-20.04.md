---
layout: post
title: Kernel Livepatch in Ubuntu Pro 20.04
---

In this article, I'll explain how to see which live patches you get on Ubuntu Pro with the Kernel Livepatch feature from Canonical. With [Canonical's Livepatch service](https://ubuntu.com/security/livepatch), kernel patches are delivered immediately without the need to reboot your VMs.

Ubuntu Pro on  [AWS](https://ubuntu.com/aws/pro)  and  [Azure](https://ubuntu.com/azure/pro)  (and now on  [Google Cloud](https://ubuntu.com/gcp/pro)  too!) comes with Livepatch enabled out of the box.

The first step is to launch an Ubuntu Pro 20.04 instance. I ran the following steps on an Ubuntu Pro 20.04 instance in Azure. You can find the different Ubuntu Pro releases in the marketplace of each cloud.

After launching an Ubuntu Pro 20.04 instance, check the Ubuntu and kernel versions:

```console
ubuntu@focalpro:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.2 LTS
Release:	20.04
Codename:	focal

ubuntu@focalpro:~$ uname -a
Linux focalpro 5.4.0-1043-azure #45-Ubuntu SMP Fri Mar 19 17:33:38 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

You can run `canonical-livepatch status` (I like to specify `--format yaml` to get more output) to see the status of livepatch. It's possible that you won't see any live patches if you launched a recent image of Ubuntu Pro:

```console
ubuntu@focalpro:~$ canonical-livepatch status --format yaml
client-version: 9.5.5
architecture: x86_64
cpu-model: Intel(R) Xeon(R) CPU E5-2673 v4 @ 2.30GHz
last-check: 2021-04-08T17:35:44Z
boot-time: 2021-04-08T17:24:51Z
uptime: 11m7s
status:
- kernel: 5.4.0-1043.45-azure
  running: true
  livepatch:
    checkState: checked
    patchState: nothing-to-apply
    version: ""
    fixes: ""
```

But we want to see some patches, right? So let's downgrade the kernel and find out what happens when there are patches to enable.

On Azure, start by listing the possible kernel versions of your currently running kernel branch (5.4.0 in this example):

```console
ubuntu@focalpro:~$ apt-cache search linux-image | grep 5.4.0- | grep azure
[...]
linux-image-5.4.0-1010-azure - Signed kernel image azure
linux-image-5.4.0-1012-azure - Signed kernel image azure
linux-image-5.4.0-1016-azure - Signed kernel image azure
linux-image-5.4.0-1019-azure - Signed kernel image azure
linux-image-5.4.0-1020-azure - Signed kernel image azure
linux-image-5.4.0-1022-azure - Signed kernel image azure
linux-image-5.4.0-1023-azure - Signed kernel image azure
linux-image-5.4.0-1025-azure - Signed kernel image azure
linux-image-5.4.0-1026-azure - Signed kernel image azure
linux-image-5.4.0-1031-azure - Signed kernel image azure
linux-image-5.4.0-1034-azure - Signed kernel image azure
linux-image-5.4.0-1035-azure - Signed kernel image azure
linux-image-5.4.0-1036-azure - Signed kernel image azure
linux-image-5.4.0-1039-azure - Signed kernel image azure
linux-image-5.4.0-1040-azure - Signed kernel image azure
linux-image-5.4.0-1041-azure - Signed kernel image azure
linux-image-5.4.0-1043-azure - Signed kernel image azure
[...]
```

I chose to install the 5.4.0-1012 package which was released in May 2020:

```console
ubuntu@focalpro:~$ sudo apt install linux-azure-cloud-tools-5.4.0-1012 linux-azure-headers-5.4.0-1012 linux-azure-tools-5.4.0-1012 linux-cloud-tools-5.4.0-1012-azure linux-headers-5.4.0-1012-azure linux-image-5.4.0-1012-azure linux-modules-5.4.0-1012-azure linux-tools-5.4.0-1012-azure
```

Now you need to reboot to use that kernel. I find it easier for testing purposes to just remove the newer kernels to force the VM to boot with the older kernel, but I wouldn't recommend doing that on an important VM since you might be stuck with no bootable kernel:

```console
ubuntu@focalpro:~$ sudo apt purge linux-azure-cloud-tools-5.4.0-1043 linux-azure-headers-5.4.0-1043 linux-azure-tools-5.4.0-1043 linux-cloud-tools-5.4.0-1043-azure linux-headers-5.4.0-1043-azure linux-image-5.4.0-1043-azure linux-modules-5.4.0-1043-azure linux-tools-5.4.0-1043-azure
```

After the reboot, check that you're really running the older kernel:

```console
ubuntu@focalpro:~$ uname -a
Linux focalpro 5.4.0-1012-azure #12-Ubuntu SMP Mon May 11 13:30:06 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

And now look at the status of livepatch:

```console
ubuntu@focalpro:~$ canonical-livepatch status
last check: 52 seconds ago
kernel: 5.4.0-1012.12-azure
server check-in: succeeded
patch state: ✓ all applicable livepatch modules inserted
patch version: 75.2

ubuntu@focalpro:~$ canonical-livepatch status --format yaml
client-version: 9.5.5
architecture: x86_64
cpu-model: Intel(R) Xeon(R) CPU E5-2673 v4 @ 2.30GHz
last-check: 2021-04-08T17:41:06Z
boot-time: 2021-04-08T17:40:46Z
uptime: 1m0s
status:
- kernel: 5.4.0-1012.12-azure
  running: true
  livepatch:
    checkState: checked
    patchState: applied
    version: "75.2"
    fixes: |-
      * CVE-2013-1798
      * CVE-2019-0155
      * CVE-2019-0155: LP: #1852141
      * CVE-2019-14615
      * CVE-2019-14895
      * CVE-2019-14896
      * CVE-2019-14897
      * CVE-2019-14901
      * CVE-2019-18885
      * CVE-2019-19642
      * CVE-2019-20096
      * CVE-2019-3016
      * CVE-2020-10757
      * CVE-2020-11494
      * CVE-2020-11935
      * CVE-2020-12114
      * CVE-2020-12351
      * CVE-2020-12352
      * CVE-2020-14386
      * CVE-2020-14416
      * CVE-2020-16119
      * CVE-2020-16120
      * CVE-2020-24490
      * CVE-2020-27170
      * CVE-2020-27171
      * CVE-2020-2732
      * CVE-2020-28374
      * CVE-2020-8647
      * CVE-2020-8648
      * CVE-2020-8649
      * CVE-2021-27363
      * CVE-2021-27364
      * CVE-2021-27365
      * CVE-2021-3444
```

We can see that we're running the version 75.2 of the livepatch bundle, which corresponds to the [latest livepatch security notice](https://lists.ubuntu.com/archives/ubuntu-security-announce/2021-April/005960.html) in the USN mailing list. This bundle protects this kernel from 33 CVEs.

That mailing list is where you can find the information about each individual livepatch. If you start from the [Ubuntu security announce mailing list archive](https://lists.ubuntu.com/archives/ubuntu-security-announce/), you can click on each month (I like to use the Date indices) and find the notices that have LSN instead of USN. 

Here's a quick list of all the LSN notices as of this writing:

```
Subject: [LSN-0012-1] Linux kernel vulnerability     Date: Thu, 20 Oct 2016 13:49:35 -0000
Subject: [LSN-0013-1] Linux kernel vulnerability     Date: Wed, 30 Nov 2016 18:17:02 -0000
Subject: [LSN-0014-1] Linux kernel vulnerability     Date: Wed, 07 Dec 2016 11:19:59 -0000
Subject: [LSN-0021-1] Linux kernel vulnerability     Date: Thu, 13 Apr 2017 14:36:25 -0400
Subject: [LSN-0022-1] Linux kernel vulnerability     Date: Tue, 16 May 2017 17:44:55 -0400
Subject: [LSN-0024-1] Linux kernel vulnerability     Date: Wed, 21 Jun 2017 18:56:50 -0400
Subject: [LSN-0025-1] Linux kernel vulnerability     Date: Thu, 06 Jul 2017 10:55:15 -0400
Subject: [LSN-0026-1] Linux kernel vulnerability     Date: Mon, 24 Jul 2017 16:44:10 -0400
Subject: [LSN-0027-1] Linux kernel vulnerability     Date: Thu, 03 Aug 2017 15:57:30 -0400
Subject: [LSN-0028-1] Linux kernel vulnerability     Date: Thu, 17 Aug 2017 14:05:00 -0400
Subject: [LSN-0029-1] Linux kernel vulnerability     Date: Wed, 30 Aug 2017 09:15:56 -0400
Subject: [LSN-0030-1] Linux kernel vulnerability     Date: Mon, 18 Sep 2017 14:50:46 -0400
Subject: [LSN-0031-1] Linux kernel vulnerability     Date: Tue, 10 Oct 2017 11:18:20 -0400
Subject: [LSN-00X32-1] Linux kernel vulnerability    Date: Tue, 21 Nov 2017 10:46:03 -0500
Subject: [LSN-0032-2] Linux kernel vulnerability     Date: Thu, 23 Nov 2017 12:04:14 -0500
Subject: [LSN-0033-1] Linux kernel vulnerability     Date: Thu, 07 Dec 2017 20:50:57 -0500
Subject: [LSN-0034-1] Linux kernel vulnerability     Date: Tue, 09 Jan 2018 18:36:23 -0500
Subject: [LSN-0035-1] Linux kernel vulnerability     Date: Thu, 22 Feb 2018 21:15:00 -0500
Subject: [LSN-0036-1] Linux kernel vulnerability     Date: Mon, 02 Apr 2018 09:54:49 -0400
Subject: [LSN-0037-1] Linux kernel vulnerability     Date: Wed, 02 May 2018 10:28:33 -0400
Subject: [LSN-0038-1] Linux kernel vulnerability     Date: Tue, 08 May 2018 22:24:56 -0400
Subject: [LSN-0039-1] Linux kernel vulnerability     Date: Fri, 25 May 2018 10:33:58 -0400
Subject: [LSN-0040-1] Linux kernel vulnerability     Date: Tue, 03 Jul 2018 10:33:29 -0400
Subject: [LSN-0041-1] Linux kernel vulnerability     Date: Fri, 10 Aug 2018 07:32:20 -0400
Subject: [LSN-0042-1] Linux kernel vulnerability     Date: Tue, 14 Aug 2018 14:46:13 -0400
Subject: [LSN-0043-1] Linux kernel vulnerability     Date: Tue, 11 Sep 2018 07:46:26 -0400
Subject: [LSN-0044-1] Linux kernel vulnerability     Date: Mon, 08 Oct 2018 07:28:54 -0400
Subject: [LSN-0045-1] Linux kernel vulnerability     Date: Wed, 14 Nov 2018 09:34:09 -0500
Subject: [LSN-0046-1] Linux kernel vulnerability     Date: Thu, 20 Dec 2018 10:26:55 -0500
Subject: [LSN-0051-1] Linux kernel vulnerability     Date: Tue, 14 May 2019 13:59:52 -0400
Subject: [LSN-0052-1] Linux kernel vulnerability     Date: Tue, 18 Jun 2019 08:52:11 -0400
Subject: [LSN-0053-1] Linux kernel vulnerability     Date: Mon, 29 Jul 2019 10:50:14 -0400
Subject: [LSN-0054-1] Linux kernel vulnerability     Date: Wed, 28 Aug 2019 13:18:23 -0400
Subject: [LSN-0055-1] Linux kernel vulnerability     Date: Fri, 06 Sep 2019 08:54:02 -0400
Subject: [LSN-0056-1] Linux kernel vulnerability     Date: Fri, 20 Sep 2019 16:35:43 +0200
Subject: [LSN-0058-1] Linux kernel vulnerability     Date: Tue, 22 Oct 2019 08:16:38 -0400
Subject: [LSN-0059-1] Linux kernel vulnerability     Date: Tue, 12 Nov 2019 18:32:29 -0500
Subject: [LSN-0061-1] Linux kernel vulnerability     Date: Wed, 08 Jan 2020 08:10:46 -0500
Subject: [LSN-0062-1] Linux kernel vulnerability     Date: Mon, 03 Feb 2020 08:10:44 -0500
Subject: [LSN-0063-1] Linux kernel vulnerability     Date: Wed, 19 Feb 2020 10:46:52 -0500
Subject: [LSN-0064-1] Linux kernel vulnerability     Date: Thu, 19 Mar 2020 08:23:33 -0400
Subject: [LSN-0065-1] Linux kernel vulnerability     Date: Thu, 09 Apr 2020 10:20:57 -0400
Subject: [LSN-0071-1] linux kernel vulnerability     Date: Thu, 10 Sep 2020 23:20:34 -0400
Subject: [LSN-0072-1] linux kernel vulnerability     Date: Wed, 14 Oct 2020 12:20:35 -0400
Subject: [LSN-0073-1] Linux kernel vulnerability     Date: Fri, 23 Oct 2020 09:44:40 -0400
Subject: [LSN-0074-1] Linux kernel vulnerability     Date: Mon, 01 Feb 2021 10:26:52 -0500
Subject: [LSN-0075-1] Linux kernel vulnerability     Date: Wed, 07 Apr 2021 12:20:38 -0400
```

**NOTE:** Recently, there was a defective livepatch for kernel 4.4 in Ubuntu 16.04 LTS (Xenial) that was not caught in the Canonical internal testing processes because the defect was a race condition, triggered by workload-specific behaviour, under load. [Canonical has issued an incident report](https://ubuntu.com/blog/livepatch-2021-03-24-incident-investigation-report) in which they do a root cause analysis as well as describe improvements they're doing to the process to catch issues like this one in the future.

