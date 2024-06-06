# meta-xt-prod-devel-rpi5

The Xen-Troops RPI 5 Xen demonstration project is intended to demonstrate how RPI 5 can be used
to run with Xen which includes the following images:

* Zephyr operated control domain which is a thin Zephyr Dom0 `zephyr-dom0-xt` application
  with shell and SD-card storage support (SDHC). It also supports dom0less functionality.
  This domain is resposible for Xen domains control.
  It includes xen-tools library which gives an ability to control other domains.
* Linux operated driver domain. This domain managing hardware and provides access to
  some of this hardware to other domains using PV backends. It has access to USB and rootfs
  is placed on USB storage.
* A set of domains to be started from the control domain.

Once Zephyr control domain booted it can start other guest domains.

Boot process:

* RPI5 bootloder started **u-boot**
* **u-boot** executes boot script:
  * loads **Zephyr Dom0** binary
  * loads **RPI 5 Linux DomU** binary for DomU0
  * loads **Xen Passthrough device tree** binary for DomU0
  * loads **Linux System device tree device tree** binary
  * applies changes to **Linux System device tree device tree** for Xen `dom0less` boot mode
  * loads **Xen** binary and starts **Xen**
* **Xen** is booted and starts **Zephyr Dom0 zephyr-dom0-xt** and RPI 5 Linux DomU0

Build for the driver domain device is based on RPI5 bsp yocto build:
[meta-raspberrypi](https://git.yoctoproject.org/meta-raspberrypi)

## Status

This is release 0.1.0. This release supports the following features:

* Zephyr operated control domain:
  * xen libraries integration that allows control of the other domains.
  * domd with running OCI containers as Aos services capability.
* Linux operated driver domain:
  * controls hardware;
  * provides PV backends for the domains.
* rpi_5_domd domain
  * zephyr-based domain built on top of zephyr-blinky sample;
  * GPIO device passedthrough to the domain to control onboard LED;
  * do not start automatically.
* rpi_5_domu domain
  * zephyr-based domain built on top of zephyr-sync sample;
  * do not start automatically.
* helloworld_xen-arm64 domain:
  * demonstrates work of unikernel images as a domains.
* linux_pv_domu domain:
  * Simple linux-based domain with initramfs
  * implements PVnet interface to access other domains via pv network;
  * impelments PVblock interface which allows to mount block devices to the domains.

Testing pvnet connection instruction:
[linux_pv_domu testing](https://github.com/xen-troops/meta-xt-rpi5/wiki/RPI5-build-Linux-domain#test)

## System Requirements

Below dependencies have to be satisfied before start building project:

[Moulin requirements](https://moulin.readthedocs.io/en/latest/about.html#requirements-and-installation)

[Yocto requirements](https://docs.yoctoproject.org/ref-manual/system-requirements.html)

[Zephyr requirements](https://docs.zephyrproject.org/latest/develop/getting_started/index.html)

## Build

```
mkdir <my_build_dir>
cd <my_build_dir>
curl -O https://raw.githubusercontent.com/xen-troops/meta-xt-prod-devel-rpi5/master/rpi5.yaml
moulin rpi5.yaml
ninja
```

## Create sd image

```
ninja full.img
```
or
```
ninja full.img.bmap
```
or
```
ninja full.img.gz
```

In case of use domd rootfs from the usb flash drive rootfs partition will not be created in image. Rootfs
image should be written separately to the appropriate partition on media witbh command:
```
dd if=yocto/build-domd/tmp/deploy/images/raspberrypi5/rpi5-image-xt-domd-raspberrypi5.rootfs.ext4 of=<partition device> bs=1M
```
Note: before writing rootfs media should be properly partitioned for example with `fdisk` command

## Testing

This section shows the way to start domains:

To start domains please execute the following commands from
control domain console:

* rpi_5_domd
```
	xu create rpi_5_domd
	xu console <domain id>
```

* rpi_5_domu

```
	xu create rpi_5_domu
	xu console <domain id>
```

* helloworld_xen-arm64

```
	xu create helloworld_xen-arm64
	xu console <domain id>
```

* linux_pv_domu

```
	xu create linux_pv_domu
	xu console <domain id>
```

To switch between control domain and driver domain consoles
please press "Ctrl + A" 6 times.

Current domain id can be get by using command:
```
	xstat stat
```

after starting the domain from control domain console.

## Start Zephyr rpi_5_domd domain example

TODO: cmds and logs

## Start Zephyr rpi_5_domu domain example

TODO: cmds and logs

## Start Unikraft helloworld_xen-arm64 domain example

TODO: cmds and logs

## Start Lunux linux_pv_domu domain example

TODO: cmds and logs

## Bootlog

TODO

## Wiki
Link to the wiki pages:
[wiki](https://github.com/xen-troops/meta-xt-rpi5/wiki)

## Known issues
1. Sometime rpi boot firmware files are not deployed during build. In this case rpi-bootfiles, rpi-config and
rpi-cmdline recipes should be cleaned with yocto command:
```
cd yocto
. poky/oe-init-build-env build-domd
bitbake -c clean rpi-bootfiles -c clean rpi-config -c clean rpi-cmdline
cd ../..
ninja
```
