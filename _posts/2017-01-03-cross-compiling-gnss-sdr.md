---
title: "Cross-compiling GNSS-SDR"
permalink: /docs/tutorials/cross-compiling/
excerpt: "A guide to cross-compile GNSS-SDR for embedded platforms."
author_profile: false
header:
  teaser: /assets/images/oe-logo.png
tags:
  - tutorial
  - embedded
sidebar:
  nav: "docs"
toc: true
last_modified_at: 2017-11-23T09:37:02+02:00
---

An **embedded system** is defined as a computer system with a specific function within a larger mechanical or electrical system. Examples of properties of embedded computers when compared with general-purpose counterparts are low power consumption, small size, rugged operating ranges, and low per-unit cost, at the price of limited processing resources.

This page is devoted to the development cycle for building and executing GNSS-SDR in an embedded computer. In this example we are working with a [Zedboard](https://www.xilinx.com/products/boards-and-kits/1-elhabt.html) (a development board that ships a [Xilinx Zynq-7000](https://www.xilinx.com/products/silicon-devices/soc/zynq-7000.html) all-programmable [SoC](https://en.wikipedia.org/wiki/System_on_a_chip), which houses two ARM and one FPGA processor in a single chip), but this procedure is applicable to other embedded platforms without much modification.

Once all the required dependencies are already installed, GNSS-SDR can be built from source in ARM processors without hassle. However, this building process can easily take more than 10 hours if it is executed on the Zynq device. Thus, in order to speed up the development cycle from a change in the source code to the execution in an embedded platform, we need to resort to cross-compilation.

**Cross-compilation** consists of a building framework capable of creating executable code for a platform other than the one on which the compiler is running. In our example, we would like to build GNSS-SDR with the powerful, fast processor of a general-purpose desktop computer, and to generate binaries that can be directly executed by the Zynq device.

  By using cross-compilation, we can shorten the building time from more than 10 hours to less than 10 minutes. This improves [**Testability**]({{ "/design-forces/testability/" | absolute_url }}), as one of its requirements is that a testing cycle has to be *fast*.
  {: .notice--success}

The cross-compilation environment proposed here is based on [OpenEmbedded](http://www.openembedded.org), a building framework for embedded Linux. OpenEmbedded offers a best-in-class cross-compile environment, allowing developers to create a complete, custom GNU/Linux distribution for embedded systems.

Below we provide a software developer kit (SDK) that installs a ready-to-use cross-compilation environment in your computer.

Getting the SDK
----------------

We offer two options here: you can either download a script that will install the full SDK in your computer, or you can customise and build your own SDK. Both options are described below:

### Option 1: Downloading the SDK

You can download the SDK from the links below. Version names (Jethro, Krogoth, Morty, ...) follow those of the [Yocto Project Releases](https://wiki.yoctoproject.org/wiki/Releases).

The following table lists the available SDK versions:


| Version | Status | Download | md5 | Manifest |
|:-|:-:|:-:|:-|:-:|
| Rocko | Stable | [SDK](http://sites.cttc.es/gnss_files/SDK/Rocko/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh){:download="oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh"}  | 3a59b721a1b018ce445366ba859a5988 | [Host](http://sites.cttc.es/gnss_files/SDK/Rocko/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.host.manifest), [Target](http://sites.cttc.es/gnss_files/SDK/Rocko/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.target.manifest) |
| Pyro | Stable | [SDK](http://sites.cttc.es/gnss_files/SDK/Pyro/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh){:download="oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh"} | 3a220123b367ac8a5ba39417c71aec68 | [Host](http://sites.cttc.es/gnss_files/SDK/Pyro/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.host.manifest), [Target](http://sites.cttc.es/gnss_files/SDK/Pyro/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.target.manifest) |
| Morty | Stable | [SDK](http://sites.cttc.es/gnss_files/SDK/Morty/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh){:download="oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh"} | df527c8cc74aa306dfb75a81860a36f2 | [Host](http://sites.cttc.es/gnss_files/SDK/Morty/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.host.manifest), [Target](http://sites.cttc.es/gnss_files/SDK/Morty/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.target.manifest) |
| Krogoth | Stable | [SDK](http://sites.cttc.es/gnss_files/SDK/Krogoth/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh){:download="oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh"} | 3a220123b367ac8a5ba39417c71aec68 | [Host](http://sites.cttc.es/gnss_files/SDK/Krogoth/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.host.manifest), [Target](http://sites.cttc.es/gnss_files/SDK/Krogoth/oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.target.manifest) |
| Jethro | Stable | [SDK](http://sites.cttc.es/gnss_files/SDK/Jethro/oecore-x86_64-armv7ahf-vfp-neon-toolchain-nodistro.0.sh){:download="oecore-x86_64-armv7ahf-vfp-neon-toolchain-nodistro.0.sh"} | d0419e9c1e0894a327af4d9560cf0294 | [Host](http://sites.cttc.es/gnss_files/SDK/Jethro/oecore-x86_64-armv7ahf-vfp-neon-toolchain-nodistro.0.host.manifest), [Target](http://sites.cttc.es/gnss_files/SDK/Jethro/oecore-x86_64-armv7ahf-vfp-neon-toolchain-nodistro.0.target.manifest) |



Please note that the SDK scripts provided in this table take about 1.5 GB. Check out the manifest files to see the full list of packages and versions each SDK will install in the root filesystem of your device. Releases are listed from the most recent (top) to the oldest (bottom). All the SDKs include all the required dependency packages for cross-compiling GNSS-SDR in your own machine, including optional packages such as `gr-osmosdr` and `gr-iio`.


### Option 2: Building your own SDK

Head to [https://github.com/carlesfernandez/oe-gnss-sdr-manifest](https://github.com/carlesfernandez/oe-gnss-sdr-manifest) and follow instructions there. Make sure you have plenty of space in your hard drive (25 GB minimum). In summary, the process is as follows:

1) Install ```repo```:

```bash
$ curl http://storage.googleapis.com/git-repo-downloads/repo > repo
$ chmod a+x repo
$ sudo mv repo /usr/local/bin/
```

2) Create a folder in which all the process will take place:

```bash
$ mkdir oe-repo
$ cd oe-repo
```

3) Initialize ```repo```, download the required tools and prepare your building environment:

```bash
$ repo init -u git://github.com/carlesfernandez/oe-gnss-sdr-manifest.git -b krogoth
$ repo sync
$ TEMPLATECONF=`pwd`/meta-gnss-sdr/conf source ./oe-core/oe-init-build-env ./build ./bitbake
```

This last command copies default configuration information into the ```./build/conf``` directory and sets up some environment variables for OpenEmbedded.

{% capture branches_info %}
Please note that the name of the oe-gnss-sdr-manifest branch passed to ```repo``` will determine the version of the SDK to be built. For instance,

```bash
$ repo init -u git://github.com/carlesfernandez/oe-gnss-sdr-manifest.git -b jethro
```

will generate the Jethro release of the SDK (see the manifest for a list of installed packages and their respective versions), while

```bash
$ repo init -u git://github.com/carlesfernandez/oe-gnss-sdr-manifest.git -b rocko
```

will generate the Rocko release.
{% endcapture %}

<div class="notice--warning">
  {{ branches_info | markdownify }}
</div>


4) OPTIONAL: at this point, you can configure your building by editing the file ```./conf/conf.local```. If you do nothing and leave the configuration by default, the next step will generate an image for a Zedboard. Other platforms can be selected by changing the value of the MACHINE variable. Read the comments at ```./conf/conf.local``` for more options.

5) Build the image and the toolchain installer:

```bash
$ bitbake gnss-sdr-dev-image
$ bitbake -c populate_sdk gnss-sdr-dev-image
```

This process downloads several gigabytes of source code and then proceeds to compile all the required packages for the host and native targets, so **it will take time**. The first command constructs a complete Linux image for your target device. The second command generates the toolchain installer, a script that installs a cross-compiler, a cross-linker and a cross-debugger, forming a completely self-contained toolchain which allows you to cross-develop on the host machine for the target hardware. The generated script will be found under ```./tmp-glibc/deploy/sdk/```.


Using the SDK
--------------

### Installing the SDK

Download the SDK shell script (or use a locally created SDK, as explained above) and install it:

```bash
$ sudo sh oecore-x86_64-armv7ahf-neon-toolchain-nodistro.0.sh
```

This will ask you what directory to install the SDK into. Which directory does not matter, just make sure wherever it goes that you have enough disk space. The default is ```/usr/local```.

The SDK comes with everything you need to build GNSS-SDR. The main contents it has are:

* An "```environment-setup-...```" script that sets up our environmental variables, like editing PATH, CC, CXX, etc.
* Two sysroots; one for the host machine and one for the target device (installed by default at ```/usr/local/oecore-x86_64/sysroots/```).

### Setting up the cross-compiling environment
Running the environment script will set up most of the variables you'll need to compile. You will need to do this each time you want to run the SDK (and since the environment variables are only set for the current shell, you need to source it for every console you will run the SDK from):

```bash
$ . /usr/local/oecore-x86_64/environment-setup-armv7ahf-neon-oe-linux-gnueabi
```

### Cross-compiling GNSS-SDR and installing it on the target filesystem

Once the environment script has been run, you can cross-compile GNSS-SDR as:

```bash
$ git clone https://github.com/gnss-sdr/gnss-sdr.git
$ cd gnss-sdr
$ git checkout next
$ cd build
$ cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/Toolchains/oe-sdk_cross.cmake -DCMAKE_INSTALL_PREFIX=/usr ..
$ make
$ sudo make install DESTDIR=/usr/local/oecore-x86_64/sysroots/armv7ahf-neon-oe-linux-gnueabi/
```

Please note that we set the install prefix to ```/usr```. That will be the installation location of the project on the embedded device. We use this because all links and references within the file system will be based on this prefix, but it is obviously not where we want to install these files on our own host system. Instead, we use the ```make``` program's ```DESTDIR``` directive. On the device itself, however, the file system would have this installed onto ```/usr```, which means all our links and references are correct as far as the device is concerned.



Copying an image file to your SD card
-------------------------------------

We have several options here:

### Using ```dd```

```bash
$ mkdir myimage
$ tar -xvzf gnss-sdr-dev-image-zedboard-zynq7-20170103150322.rootfs.tar.gz -C myimage
$ sudo dd status=progress bs=4M if=myimage of=/dev/sdX
```

where ```/dev/sdX``` is the device the card is mounted as. This works, but can be slow.

### Using ```bmaptool```

This option is faster:

```bash
$ git clone https://github.com/01org/bmap-tools.git
$ cd bmap-tools
$ sudo python setup.py install
$ sudo bmaptool copy gnss-sdr-dev-image-zedboard-zynq7-20170103150322.rootfs.tar.gz /dev/sdX --nobmap
```

### Copying only the sysroot to the SD card using ```cp```

For systems with a dedicated u-boot, devicetree and Kernel, it is possible to copy only the cross-compiled sysroot to the SD ext4 partition. Mount the SD card partition and extract the root filesystem to the mounted root directory (in this example, ```sdb2``` is the SD card device and the ext4 partition is the second partition in the SD partition table), and then use ```cp``` with the ```-a``` option, which preserves the same directory tree, same file types, same contents, same metadata (times, permissions, extended attributes, etc.) and same symbolic links:

```bash
$ mkdir ./mounted_SD
$ sudo mount -rw /dev/sdb2 ./mounted_SD
$ cd ./mounted_SD
$ sudo rm -rf *
$ cd ..
$ sudo cp /usr/local/oecore-x86_64/sysroots/armv7ahf-neon-oe-linux-gnueabi/* -a ./mounted_SD
```

### Copying only GNSS-SDR executables to the device over the network using ```sshfs```

For example, let's assume that we can address the device by a network name or IP address. Let's say it's called "mydevice" and it has an ip address of 192.168.2.2. We would use a mount point created in your home directory. To install sshfs and mount mydevice locally:

```bash
$ sudo apt-get install sshfs
$ sudo gpasswd -a $USER fuse
$ cd
$ mkdir mydevice
$ sshfs -o allow_root root@192.168.2.2:/ mydevice
```

You should be able to ```ls mydevice``` and see the contents of mydevice's file system. Then you can cross-compile GNSS-SDR as before, changing the last command by:

```bash
$ sudo make install DESTDIR=~/mydevice
```

in order to install the GNSS-SDR binary directly in your device. To unmount:

```bash
$ fusermount -u ~/mydevice
```

References
--------

 * More information about the development environment and the usage of BitBake can be found in the [Yocto Project Documentation](https://www.yoctoproject.org/documentation).

 * This work is heavily based on [Embedded Developments with GNU Radio](https://wiki.gnuradio.org/index.php/Embedded_Development_with_GNU_Radio) and the work by Philip Balister (and others) on the [oe-gnuradio-manifest](https://github.com/balister/oe-gnuradio-manifest) and the [meta-sdr](https://github.com/balister/meta-sdr) layer.
