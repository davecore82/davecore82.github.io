---
layout: post
title: Installing Kafka on Ubuntu Pro with ESM Apps
---

This article goes into the details of one of the benefits of Ubuntu Pro, namely ESM Apps. We'll install Kafka on Ubuntu Pro 18.04 right from the ESM Apps repository, something that you normally can't do this easily. 

The [Enterprise open source support page](https://ubuntu.com/support) on the Ubuntu website says this about Ubuntu Advantage for Applications:

> Security patching for over 28,000 packages in Ubuntu Universe and Main

That page lists Kafka among some of the applications that are covered by Ubuntu Advantage Applications.

If you look at the [install instructions from the Apache Kafka website](https://kafka.apache.org/quickstart), you can see you have to download the tarball and install Kafka manually.  Those instructions would definitely work, but you'll end up with a manually downloaded Kafka that doesn't stay updated as new versions come out.  ESM Apps comes out of the box with Ubuntu Pro and make it easy to install Kafka on Ubuntu AND keep it updated as new security patches come out. 

So let's install Kafka on Ubuntu Pro 18.04 and find out what that means. Ubuntu Pro on  [AWS](https://ubuntu.com/aws/pro)  and  [Azure](https://ubuntu.com/azure/pro)  (and now on  [Google Cloud](https://ubuntu.com/gcp/pro)  too!) comes with the Ubuntu Advantage client that allows users to enable features of  [Ubuntu Advantage](https://ubuntu.com/advantage).

The first step is to launch an Ubuntu Pro 18.04 instance. I tested this procedure on Azure. You can find the different Ubuntu Pro releases in the marketplace of each cloud.

Once logged in to your Ubuntu Pro 18.04 instance, you can look at the status of the UA client:
```console
ubuntu@bionicpro:~$ ua status
SERVICE       ENTITLED  STATUS    DESCRIPTION
esm-apps      yes       enabled   UA Apps: Extended Security Maintenance (ESM)
esm-infra     yes       enabled   UA Infra: Extended Security Maintenance (ESM)
fips          yes       disabled  NIST-certified FIPS modules
fips-updates  yes       disabled  Uncertified security updates to FIPS modules
livepatch     yes       enabled   Canonical Livepatch service
```
It's the esm-apps feature that we're interested in here. 

So let's update the apt database and check where we can kafka from:

```console
ubuntu@bionicpro:~$ sudo apt update
ubuntu@bionicpro:~$ apt-cache policy kafka
kafka:
  Installed: (none)
  Candidate: 2.2.2u1-0ubuntu1+esm2
  Version table:
     2.2.2u1-0ubuntu1+esm2 500
        500 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 Packages
```

As you can see, the only repo we can get kafka from is esm-apps. This means Kafka is not even usually available in Ubuntu. 

Let's go ahead and install kafka, which will pull it from the ESM Apps repository automatically:

```console
ubuntu@bionicpro:~$ sudo apt install kafka
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  linux-headers-4.15.0-140
Use 'sudo apt autoremove' to remove it.
The following additional packages will be installed:
  ca-certificates-java default-jre-headless fontconfig-config fonts-dejavu-core java-common libasound2 libasound2-data libfontconfig1 libgraphite2-3 libharfbuzz0b libjline-java libjpeg-turbo8 libjpeg8 liblcms2-2 liblog4j1.2-java libnspr4 libnss3 libpcsclite1 libslf4j-java libxerces2-java
  libxi6 libxml-commons-external-java libxml-commons-resolver1.1-java libxrender1 libxtst6 libzookeeper-java openjdk-11-jre-headless x11-common zookeeper zookeeperd
Suggested packages:
  default-jre libasound2-plugins alsa-utils libjline-java-doc liblcms2-utils liblog4j1.2-java-doc libmail-java pcscd libcommons-logging-java libxerces2-java-doc libxml-commons-resolver1.1-java-doc libzookeeper-java-doc libnss-mdns fonts-dejavu-extra fonts-ipafont-gothic
  fonts-ipafont-mincho fonts-wqy-microhei | fonts-wqy-zenhei fonts-indic
The following NEW packages will be installed:
  ca-certificates-java default-jre-headless fontconfig-config fonts-dejavu-core java-common kafka libasound2 libasound2-data libfontconfig1 libgraphite2-3 libharfbuzz0b libjline-java libjpeg-turbo8 libjpeg8 liblcms2-2 liblog4j1.2-java libnspr4 libnss3 libpcsclite1 libslf4j-java
  libxerces2-java libxi6 libxml-commons-external-java libxml-commons-resolver1.1-java libxrender1 libxtst6 libzookeeper-java openjdk-11-jre-headless x11-common zookeeper zookeeperd
0 upgraded, 31 newly installed, 0 to remove and 14 not upgraded.
Need to get 95.7 MB of archives.
After this operation, 245 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
Get:1 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libjpeg-turbo8 amd64 1.5.2-0ubuntu5.18.04.4 [110 kB]
Get:2 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 java-common all 0.68ubuntu1~18.04.1 [14.5 kB]
Get:3 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 liblcms2-2 amd64 2.9-1ubuntu0.1 [139 kB]
Get:4 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libjpeg8 amd64 8c-2ubuntu8 [2194 B]
Get:5 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 fonts-dejavu-core all 2.37-1 [1041 kB]
Get:6 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 fontconfig-config all 2.12.6-0ubuntu2 [55.8 kB]
Get:7 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libfontconfig1 amd64 2.12.6-0ubuntu2 [137 kB]
Get:8 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libnspr4 amd64 2:4.18-1ubuntu1 [112 kB]
Get:9 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libnss3 amd64 2:3.35-2ubuntu2.12 [1220 kB]
Get:10 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasound2-data all 1.1.3-5ubuntu0.5 [38.7 kB]
Get:11 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 libasound2 amd64 1.1.3-5ubuntu0.5 [360 kB]
Get:12 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libgraphite2-3 amd64 1.3.11-2 [78.7 kB]
Get:13 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libharfbuzz0b amd64 1.7.2-1ubuntu1 [232 kB]
Get:14 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libpcsclite1 amd64 1.8.23-1 [21.3 kB]
Get:15 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libxi6 amd64 2:1.7.9-1 [29.2 kB]
Get:16 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libxrender1 amd64 1:0.9.10-1 [18.7 kB]
Get:17 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 x11-common all 1:7.7+19ubuntu7.1 [22.5 kB]
Get:18 http://azure.archive.ubuntu.com/ubuntu bionic/main amd64 libxtst6 amd64 2:1.2.3-1 [12.8 kB]
Get:19 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 openjdk-11-jre-headless amd64 11.0.10+9-0ubuntu1~18.04 [37.5 MB]
Get:20 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 libzookeeper-java all 3.4.13-3 [1385 kB]
Get:21 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 zookeeper all 3.4.13-3 [117 kB]
Get:22 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 zookeeperd all 3.4.13-3 [14.7 kB]
Get:23 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 kafka all 2.2.2u1-0ubuntu1+esm2 [50.7 MB]
Get:24 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 default-jre-headless amd64 2:1.11-68ubuntu1~18.04.1 [10.9 kB]
Get:25 http://azure.archive.ubuntu.com/ubuntu bionic-updates/main amd64 ca-certificates-java all 20180516ubuntu1~18.04.1 [12.2 kB]
Get:26 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 libjline-java all 1.0-2 [69.4 kB]
Get:27 http://azure.archive.ubuntu.com/ubuntu bionic-updates/universe amd64 liblog4j1.2-java all 1.2.17-8+deb10u1build0.18.04.1 [435 kB]
Get:28 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 libslf4j-java all 1.7.25-3 [141 kB]
Get:29 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 libxml-commons-external-java all 1.4.01-3 [240 kB]
Get:30 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 libxml-commons-resolver1.1-java all 1.2-9 [91.1 kB]
Get:31 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 libxerces2-java all 2.11.0-8 [1344 kB]
Fetched 95.7 MB in 2s (57.7 MB/s)                                     
[...]
Running hooks in /etc/ca-certificates/update.d...

done.
done.
Processing triggers for ureadahead (0.100.0-21) ...
```

Note that 4 packages were pulled from the ESM apps repo:

```
Get:20 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 libzookeeper-java all 3.4.13-3 [1385 kB]
Get:21 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 zookeeper all 3.4.13-3 [117 kB]
Get:22 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 zookeeperd all 3.4.13-3 [14.7 kB]
Get:23 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 kafka all 2.2.2u1-0ubuntu1+esm2 [50.7 MB]
```

Let's take a deeper look at these 4 packages:

```console
ubuntu@bionicpro:~$ for package in libzookeeper-java zookeeper zookeeperd kafka; do apt-cache policy $package; done

libzookeeper-java:
  Installed: 3.4.13-3
  Candidate: 3.4.13-3
  Version table:
 *** 3.4.13-3 500
        500 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 Packages
        100 /var/lib/dpkg/status
     3.4.10-3 500
        500 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages

zookeeper:
  Installed: 3.4.13-3
  Candidate: 3.4.13-3
  Version table:
 *** 3.4.13-3 500
        500 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 Packages
        100 /var/lib/dpkg/status
     3.4.10-3 500
        500 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages

zookeeperd:
  Installed: 3.4.13-3
  Candidate: 3.4.13-3
  Version table:
 *** 3.4.13-3 500
        500 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 Packages
        100 /var/lib/dpkg/status
     3.4.10-3 500
        500 http://azure.archive.ubuntu.com/ubuntu bionic/universe amd64 Packages

kafka:
  Installed: 2.2.2u1-0ubuntu1+esm2
  Candidate: 2.2.2u1-0ubuntu1+esm2
  Version table:
 *** 2.2.2u1-0ubuntu1+esm2 500
        500 https://esm.ubuntu.com/apps/ubuntu bionic-apps-security/main amd64 Packages
        100 /var/lib/dpkg/status
```

We see again that kafka is only available in ESM Apps, but now we also see that the zookeeper packages come from both universe and ESM Apps, but that the versions in ESM Apps are more recent.

And finally let's take a look at the zookeeper package and see what patches we get with the 3.4.13 ESM Apps version that we wouldn't otherwise get if we stayed with the 3.4.10 version from universe:

```
ubuntu@bionicpro:~$ dpkg -L zookeeper | grep -i changelog
/usr/share/doc/zookeeper/changelog.Debian.gz
ubuntu@bionicpro:~$ zcat /usr/share/doc/zookeeper/changelog.Debian.gz | head -300
zookeeper (3.4.13-3) unstable; urgency=medium

  * Address FTBFS with GCC 9 (Closes: #925869)
  * Bump Standards-Version to 4.4.0
  * Freshen years in debian/copyright and update Source:

 -- tony mancill <tmancill@debian.org>  Sat, 17 Aug 2019 10:59:19 -0700

zookeeper (3.4.13-2) unstable; urgency=medium

  * Add patch for CVE-2019-0201 (Debian: #929283)

 -- tony mancill <tmancill@debian.org>  Tue, 04 Jun 2019 21:22:04 -0700

zookeeper (3.4.13-1) unstable; urgency=medium

  * New upstream version 3.4.13
  * Update version in debian-provided pom file
  * Bump Standards-Version to 4.2.1

 -- tony mancill <tmancill@debian.org>  Thu, 18 Oct 2018 20:19:42 -0700

zookeeper (3.4.12-2) unstable; urgency=medium

  * Team upload.

  [ tony mancill ]
  * Add patch for FTBFS with GCC-8 (Closes: #897892)

  [ Markus Koschany ]
  * Declare compliance with Debian Policy 4.2.0.
  * debian/control: Remove ancient X-Python-Version field because it is
    satisfied even in oldstable.
  * Add 15-javadoc-doclet.patch. The missing doclet class causes a javadoc
    error. Now the documentation is built again.

 -- Markus Koschany <apo@debian.org>  Thu, 23 Aug 2018 12:16:06 +0200

zookeeper (3.4.12-1) unstable; urgency=medium

  * Team upload.
  * New upstream release
    - Refreshed the patches
    - Added the Yetus annotations
  * No longer build the Netty based connection factory (depends on an obsolete
    version of Netty)
  * Ensure the unit tests are run in offline mode (Closes: #860650)
  * Disabled the i386 junit tests (no longer build due to dependencies changes)
  * Removed the unused build dependency on checkstyle
  * Modified debian/watch to catch all the past releases
  * Standards-Version updated to 4.1.4
  * Use salsa.debian.org Vcs-* URLs
  * Updated the upstream GPG keys

 -- Emmanuel Bourg <ebourg@apache.org>  Tue, 05 Jun 2018 23:45:32 +0200

zookeeper (3.4.10-3) unstable; urgency=medium

  * The default value of JMXLOCALONLY is now true (Closes: #869912)
  * Drop transitional package libzookeeper2 (Closes: #878994)
  * /var/lib/zookeeper is no longer world-readable (Closes: #870271)
  * Use debhelper 11 and set compat level to 11
  * Bump Standards-Version to 4.1.3
  * Use https for Homepage URL
  * Drop build-dep on dh-systemd in favor of debhelper 10
  * Remove init.d skeleton template boilerplate
  * Remove Upstart config
  * Add ./NOTICE.txt to debian/zookeeper.docs
  * Set build target to Java 8
  * Correct ivy profile during test phase on i386 (See: #860650)
    Note that this does not mean that tests pass on i386 now, only
    that the tests can be run without ivy dying.
  * Disable tests that use the MiniKdc class
  * Rename zooinspector JAR file to zookeeper-zooinspector as per policy
  * Add debian/NEWS

 -- tony mancill <tmancill@debian.org>  Sat, 03 Feb 2018 14:58:02 -0800
```


And this is just one of the apps from this new ESM Apps ecosystem. There are more apps in the portfolio now covered under UA Apps:
![apps supported by UA](https://raw.githubusercontent.com/davecore82/davecore82.github.io/master/images/apps-support.png)


