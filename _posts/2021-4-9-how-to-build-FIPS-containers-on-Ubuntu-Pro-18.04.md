---
layout: post
title: How to build FIPS containers on Ubuntu Pro 18.04
---

This article gives an example of how to build FIPS containers on Ubuntu Pro 18.04. There are probably other ways to do this but this is one way that doesn't require an Ubuntu Advantage token from Canonical. Instead we'll use the FIPS repository already enabled in Ubuntu Pro FIPS 18.04 to build the container.

Some of these steps could probably be simplified (ie. having to copy the deb packages manually inside the container at build time), but [this bug](https://github.com/canonical/ubuntu-advantage-client/issues/1441) seems to be causing issues with this automation. I think one way to simplify this is if we were able to run `ua auto-attach` inside the container and be able to install the FIPS packages more easily without having to download and copy each deb package manually like we're doing in this article.

Anyway! First, you'll want to start with an Ubuntu Pro **FIPS** 18.04 image on the public clouds. This is not the standard Ubuntu Pro 18.04 image, but the FIPS version of it that already comes with FIPS enabled out of the box (for example this [FIPS image on Azure](https://azuremarketplace.microsoft.com/en-ca/marketplace/apps/canonical.0001-com-ubuntu-pro-bionic-fips?tab=overview)). 

Once the instance is launched, create directories for your container build and install docker:
```console
$ mkdir -p ubuntu18-fips/packages
$ sudo apt update && sudo apt install -y docker.io
```

Next you'll want to download all the deb packages you need to install the FIPS packages inside your container. This list may not be the exact list you'll need for your use case, but you get the idea. This step should probably be automated in your pipeline with some kind of script that would not rely on exact version numbers. 

```console
$ sudo apt-get install -y --reinstall --download-only ubuntu-fips openssh-client openssh-client-hmac openssh-server openssh-server-hmac strongswan strongswan-hmac openssh-sftp-server libstrongswan libstrongswan-standard-plugins strongswan-starter strongswan-libcharon kcapi-tools libkcapi1 strongswan-charon openssl libssl1.1 libssl1.1-hmac

Fetched 149 MB in 3s (46.7 MB/s)           
Download complete and in download only mode

```
Next you'll want to copy those deb packages to your build directory and cd to where your Dockerfile will be:

```console
$ cp /var/cache/apt/archives/*.deb ~/ubuntu18-fips/packages/
$ cd ~/ubuntu18-fips/
```

Now create a Dockerfile that looks like this:

```console
$ vi Dockerfile 

FROM ubuntu:18.04

RUN apt update
COPY packages /tmp/
RUN apt install -y /tmp/openssh-client-hmac_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-client_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-server-hmac_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-server_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/strongswan-hmac_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/strongswan_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/kcapi-tools_1.0.3-2fips3_amd64.deb /tmp/libstrongswan-standard-plugins_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libstrongswan_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/openssh-sftp-server_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/strongswan-libcharon_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/strongswan-starter_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libkcapi1_1.0.3-2fips3_amd64.deb /tmp/strongswan-charon_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libssl1.1-hmac_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb /tmp/libssl1.1_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb /tmp/openssl_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb
RUN apt clean
RUN rm /tmp/*.deb
```

You can now build the container:

```console
$ sudo docker build -t ubuntu18-fips .
```
The output should look something like this:
```console
Sending build context to Docker daemon  149.4MB

Step 1/6 : FROM ubuntu:18.04
18.04: Pulling from library/ubuntu
6e0aa5e7af40: Pull complete 
d47239a868b3: Pull complete 
49cbb10cca85: Pull complete 
Digest: sha256:122f506735a26c0a1aff2363335412cfc4f84de38326356d31ee00c2cbe52171
Status: Downloaded newer image for ubuntu:18.04
 ---> 3339fde08fc3

Step 2/6 : RUN apt update
 ---> Running in 5bb1c1bcd2b1

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Get:1 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]
Get:3 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1402 kB]
Get:4 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]
Get:5 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]
Get:6 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [11.3 MB]
Get:7 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [2045 kB]
Get:8 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [348 kB]
Get:9 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [24.5 kB]
Get:10 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [186 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [13.5 kB]
Get:12 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1344 kB]
Get:13 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [31.4 kB]
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [2476 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [2173 kB]
Get:16 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [378 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [11.4 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [11.3 kB]
Fetched 22.3 MB in 3s (7253 kB/s)
Reading package lists...
Building dependency tree...
Reading state information...
3 packages can be upgraded. Run 'apt list --upgradable' to see them.
Removing intermediate container 5bb1c1bcd2b1
 ---> cea13efa8e4f

Step 3/6 : COPY packages /tmp/
 ---> dfffdc21c0aa

Step 4/6 : RUN apt install -y /tmp/openssh-client-hmac_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-client_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-server-hmac_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/openssh-server_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/strongswan-hmac_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/strongswan_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/kcapi-tools_1.0.3-2fips3_amd64.deb /tmp/libstrongswan-standard-plugins_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libstrongswan_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/openssh-sftp-server_1%3a7.9p1-10~ubuntu18.04.fips.0.2_amd64.deb /tmp/strongswan-libcharon_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/strongswan-starter_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libkcapi1_1.0.3-2fips3_amd64.deb /tmp/strongswan-charon_5.6.2-1ubuntu2.fips.2.4.2_amd64.deb /tmp/libssl1.1-hmac_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb /tmp/libssl1.1_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb /tmp/openssl_1.1.1-1ubuntu2.fips.2.1~18.04.3.1_amd64.deb
 ---> Running in 2018e8171aab
[...]
Running hooks in /etc/ca-certificates/update.d...
done.
Removing intermediate container 2018e8171aab
 ---> a3954c9ff148

Step 5/6 : RUN apt clean
 ---> Running in fe090fcf5f2f

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

Removing intermediate container fe090fcf5f2f
 ---> 686a8c30ef2e

Step 6/6 : RUN rm /tmp/*.deb
 ---> Running in 893520b9fc55
Removing intermediate container 893520b9fc55
 ---> 93be10a9d054
Successfully built 93be10a9d054
Successfully tagged ubuntu18-fips:latest
```

And that's it!  Now you can test drive your container with bash and confirm that FIPS is running like you expect:

```console
$ sudo docker run -it ubuntu18-fips bash

root@de88cb60565e:/# cat /proc/sys/crypto/fips_enabled 
1
root@de88cb60565e:/# dpkg -l | grep -i fips
ii  kcapi-tools                    1.0.3-2fips3                      amd64        Command-line tools for Linux Kernel Crypto API
ii  libkcapi1:amd64                1.0.3-2fips3                      amd64        Linux Kernel Crypto API User Space Interface Library
ii  libssl1.1:amd64                1.1.1-1ubuntu2.fips.2.1~18.04.3.1 amd64        Secure Sockets Layer toolkit - shared libraries
ii  libssl1.1-hmac:amd64           1.1.1-1ubuntu2.fips.2.1~18.04.3.1 amd64        Secure Sockets Layer toolkit - FIPS HMAC integrity check
ii  libstrongswan                  5.6.2-1ubuntu2.fips.2.4.2         amd64        strongSwan utility and crypto library
ii  libstrongswan-standard-plugins 5.6.2-1ubuntu2.fips.2.4.2         amd64        strongSwan utility and crypto library (standard plugins)
ii  openssh-client                 1:7.9p1-10~ubuntu18.04.fips.0.2   amd64        secure shell (SSH) client, for secure access to remote machines
ii  openssh-client-hmac:amd64      1:7.9p1-10~ubuntu18.04.fips.0.2   amd64        FIPS HMAC integrity check files for secure shell (SSH) client.
ii  openssh-server                 1:7.9p1-10~ubuntu18.04.fips.0.2   amd64        secure shell (SSH) server, for secure access from remote machines
ii  openssh-server-hmac:amd64      1:7.9p1-10~ubuntu18.04.fips.0.2   amd64        FIPS HMAC integrity check files for secure shell (SSH) server.
ii  openssh-sftp-server            1:7.9p1-10~ubuntu18.04.fips.0.2   amd64        secure shell (SSH) sftp server module, for SFTP access from remote machines
ii  openssl                        1.1.1-1ubuntu2.fips.2.1~18.04.3.1 amd64        Secure Sockets Layer toolkit - cryptographic utility
ii  strongswan                     5.6.2-1ubuntu2.fips.2.4.2         amd64        IPsec VPN solution metapackage
ii  strongswan-charon              5.6.2-1ubuntu2.fips.2.4.2         amd64        strongSwan Internet Key Exchange daemon
ii  strongswan-hmac:amd64          5.6.2-1ubuntu2.fips.2.4.2         amd64        This package provides HMAC hash files for FIPS 140-2 integrity
ii  strongswan-libcharon           5.6.2-1ubuntu2.fips.2.4.2         amd64        strongSwan charon library
ii  strongswan-starter             5.6.2-1ubuntu2.fips.2.4.2         amd64        strongSwan daemon starter and configuration file parser
```

Refer to the Ubuntu FIPS documentation to find which packages you may need for your use case. 

