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

The **rpi_5_domd**  mockups Driver Domain (DomD) behavior and uses real
RPI5 GPIO HW ``/soc/gpio@7d517c00``. It's based on Zephyr Blinky sample.

Create DomD by using ``xu create`` with **rpi_5_domd** configuration.

**NOTE**: Look for the line like **"domain:2 created"** to get Xen domain id which will be passed to
to all subsequent commands.

```
uart:~$ xu config_list
rpi_5_domd
rpi_5_domu
helloworld_xen-arm64
linux_pv_domu
uart:~$ xu create rpi_5_domd
(XEN) avc:  denied  { create } for current=d0 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { max_vcpus } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { setaddrsize } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { setdomainmaxmem } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { setpagingmempool } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { create } for current=d0 scontext=system_u:system_r:dom0_t tcontext=system_u:object_r:dom0_t tclass=event
(XEN) avc:  denied  { adjust } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=mmu
(XEN) avc:  denied  { cacheflush } for source=d0 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:dom0_t tclass=domain2
I: storage: file read /0:/dom0/z_blinky.bin size: 64
I: storage: file read /0:/dom0/z_blinky.bin size: 64
I: bootargs =
I: Extended region 0: 0x41000000->0xc0000000
I: Extended region 1: 0x200000000->0x10000000000
I: rambase = 40000000, ramsize = 16777216
I: kernbase = 40000000 kernsize = 360452, dtbsize = 8192
I: kernsize_aligned = 2097152
I: DTB will be placed on addr = 0x40e00000
I: storage: file read /0:/dom0/z_blinky.bin size: 360452
(XEN) avc:  denied  { add } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=resource
(XEN) avc:  denied  { use_noiommu } for current=d0 range=0x107d517-0x107d517 scontext=system_u:system_r:unlabeled_t tcontext=system_u:object_r:iomem_t tclass=resource
(XEN) memory_map:add: dom2 gfn=107d517 mfn=107d517 nr=1
(XEN) avc:  denied  { setvcpucontext } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { unpause } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
domain:2 created
^^^^^^^^^^^^^^^^
uart:~$ (XEN) d2v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER0
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 1 to 2 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 2 to 3 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 3 to 4 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 4 to 5 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 5 to 6 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 6 to 7 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 7 to 8 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 8 to 9 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 9 to 10 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 10 to 11 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 11 to 12 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 12 to 13 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 13 to 14 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 14 to 15 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d2v0 Expanding d2 grant table from 15 to 16 frames
(XEN) avc:  denied  { setup } for source=d2 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:unlabeled_t tclass=grant
```

At this moment RPI5 led should start blinking.

Attach to DomD console with ``xu console``, use ``Ctrl-']'`` to exit console:

```
uart:~$ xu console 2
Attached to a domain console
LED state: OFF
[00:00:00.000,000] <inf> xen_events: xen_events_init: events inited

[00:00:00.000,000] <inf> uart_hvc_xen: Xen HVC inited successfully

*** Booting Zephyr OS build cb85dfaa5099 ***
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
Detached from console
```

Pause DomD - RPI5 led should stop blinking and console should produce no new messages:

```
uart:~$ xu pause 2
(XEN) avc:  denied  { pause } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
domain:2 paused
uart:~$ xu console 2
Attached to a domain console
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
LED state: ON
LED state: OFF
Detached from console
```

Unpause DomD - RPI5 led should start blinking again:

```
uart:~$ xu unpause 2
domain:2 unpaused
```

Destroy DomD - DomD will be destroyed and Xen domain id "2" will not be accessible any more.

```
uart:~$ xu destroy 2
(XEN) avc:  denied  { destroy } for source=d0 target=d2 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
domain:2 destroyed
```

## Start Zephyr rpi_5_domu domain example

The **rpi_5_domu** mockups simple Xen guest domain (DomU) without using real RPI5 HW.
It's based on Zephyr samples/synchronization sample.

Create DomU by using ``xu create`` with **rpi_5_domu**.

**NOTE**: Look for the line like **"domain:3 created"** to get Xen domain id which will be passed to
to all subsequent commands.

```
uart:~$ xu create rpi_5_domu
I: storage: file read /0:/dom0/z_sync.bin size: 64
I: storage: file read /0:/dom0/z_sync.bin size: 64
I: bootargs =
I: Extended region 0: 0x41000000->0xc0000000
I: Extended region 1: 0x200000000->0x10000000000
I: rambase = 40000000, ramsize = 16777216
I: kernbase = 40000000 kernsize = 364548, dtbsize = 8192
I: kernsize_aligned = 2097152
I: DTB will be placed on addr = 0x40e00000
I: storage: file read /0:/dom0/z_sync.bin size: 364548
domain:3 created
^^^^^^^^^^^^^^^^
uart:~$ (XEN) d3v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER0
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 1 to 2 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 2 to 3 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 3 to 4 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 4 to 5 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 5 to 6 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 6 to 7 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 7 to 8 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 8 to 9 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 9 to 10 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 10 to 11 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 11 to 12 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 12 to 13 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 13 to 14 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 14 to 15 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d3v0 Expanding d3 grant table from 15 to 16 frames
```

Attach to DomU console with ``xu console``, use ``Ctrl-']'`` to exit console:

```
uart:~$ xu console 3
Attached to a domain console
[00:00:00.000,000] <inf> xen_events: xen_events_init: events inited

[00:00:00.000,000] <inf> uart_hvc_xen: Xen HVC inited successfully

*** Booting Zephyr OS build cb85dfaa5099 ***
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
thread_a: Hello World from cpu 0 on xenvm!
thread_b: Hello World from cpu 0 on xenvm!
Detached from console
```

Destroy domain - DomU will be destroyed and Xen domain id "3" will not be accessible any more.

```
uart:~$ xu destroy 3
domain:3 destroyed
```

## Start Unikraft helloworld_xen-arm64 domain example

The **helloworld_xen-arm64** is Xen guest domain (DomU Unikraft) without using real RPI5 HW
which is based on "Unikraft helloworld" and "monkey" examples.

Create DomU Unikraft by using ``xu create`` with **helloworld_xen-arm64**.

**NOTE**: Look for the line like **"domain:4 created"** to get Xen domain id which will be passed to
to all subsequent commands.

```
uart:~$ xu config_list
rpi_5_domd
rpi_5_domu
helloworld_xen-arm64
linux_pv_domu
uart:~$ xu create helloworld_xen-arm64
I: storage: file read /0:/dom0/helloworld_xen-arm64 size: 64
I: storage: file read /0:/dom0/helloworld_xen-arm64 size: 64
I: bootargs =
I: Extended region 0: 0x41000000->0xc0000000
I: Extended region 1: 0x200000000->0x10000000000
I: rambase = 40000000, ramsize = 16777216
I: kernbase = 40000000 kernsize = 884744, dtbsize = 8192
I: kernsize_aligned = 2097152
I: DTB will be placed on addr = 0x40e00000
I: storage: file read /0:/dom0/helloworld_xen-arm64 size: 884744
XEN) avc:  denied  { writeconsole } for current=d4 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=xen
;(d4) - Mini-OS booting -
3(d4) - Setup CPU -
2(d4) - Setup booting pagetable -
m(d4) - MMU on -
d(d4) - Setup stack -
o(d4) - Jumping to C entry -
main(XEN) d4v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER0
:4 created
^^^^^^^^^^^^^^^^
uart:~$ (XEN) xen-source/xen/common/grant_table.c:1909:d4v0 Expanding d4 grant table from 1 to 2 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d4v0 Expanding d4 grant table from 2 to 3 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d4v0 Expanding d4 grant table from 3 to 4 frames
```

Attach to DomU Unikraft console with ``xu console <domid>``, use ``Ctrl-']'`` to exit console.
There should be seen a moving monkey:

```
uart:~$ xu console 4
Attached to a domain console
Domain console overrun detected. 705 bytes was lost
00 (order 2)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc0000e0000 - ffffffc000100000 (order 5)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000100000 - ffffffc000200000 (order 8)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000200000 - ffffffc000400000 (order 9)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000400000 - ffffffc000800000 (order 10)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000800000 - ffffffc000c00000 (order 10)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000c00000 - ffffffc000e00000 (order 9)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000e00000 - ffffffc000f00000 (order 8)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000f00000 - ffffffc000f80000 (order 7)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000f80000 - ffffffc000fc0000 (order 6)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000fc0000 - ffffffc000fe0000 (order 5)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000fe0000 - ffffffc000ff0000 (order 4)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000ff0000 - ffffffc000ff8000 (order 3)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000ff8000 - ffffffc000ffc000 (order 2)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000ffc000 - ffffffc000ffe000 (order 1)
dbg:  [libukallocbbuddy] ffffffc0000d9000: Add allocate unit ffffffc000ffe000 - ffffffc000fff000 (order 0)
dbg:  [libxenplat] FDT suggests grant table base 38000000
dbg:  [libxenplat] map_gnttab, phys = 0x38000000
dbg:  [libcontext] tls_area_init: target: 0xffffffc000ffe020 (312 bytes)
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: tcb: 16 bytes
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: copy (.tdata): 0 bytes
dbg:  [libcontext] tls_area_init: uninitialized (.tbss): 0 bytes
dbg:  [libcontext] (tls_area): ffffffc000ffe020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
dbg:  [libcontext] *
dbg:  [libuksched] uk_thread 0xffffffc0000db0b0 (idle): ctx:0xffffffc0000db0b0, ectx:0xffffffc000ffc160, tlsp:0xffffffc000ffc020
dbg:  [libcontext] tls_area_init: target: 0xffffffc000ffc020 (312 bytes)
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: tcb: 16 bytes
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: copy (.tdata): 0 bytes
dbg:  [libcontext] tls_area_init: uninitialized (.tbss): 0 bytes
dbg:  [libcontext] (tls_area): ffffffc000ffc020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
dbg:  [libcontext] *
dbg:  [libcontext] ukarch_ctx 0xffffffc0000db0b0: entry:0xffffffc000049fd8(ffffffc0000db018), sp:0xffffffc000fd0020
dbg:  [libuksched] uk_thread 0xffffffc000ffd018 (init): ctx:0xffffffc000ffd018, ectx:0xffffffc000ffd0d0, tlsp:0xffffffc000ffe020
dbg:  [libukboot] Call init function: 0xffffffc00002f2cc()...
dbg:  [libukboot] Call init function: 0xffffffc00004cacc()...
dbg:  [libvfscore] (int) uk_syscall_r_dup2((int) 0x0, (int) 0x1)
dbg:  [libvfscore] (int) uk_syscall_r_dup3((int) 0x0, (int) 0x1, (int) 0x0)
dbg:  [libvfscore] (int) uk_syscall_r_dup2((int) 0x0, (int) 0x2)
dbg:  [libvfscore] (int) uk_syscall_r_dup3((int) 0x0, (int) 0x2, (int) 0x0)
dbg:  [libukboot] Call init function: 0xffffffc00004ace4()...
dbg:  [libukboot] Call init function: 0xffffffc00003b020()...
dbg:  [libukbus] Initialize bus handler 0xffffffc0000a1178...
dbg:  [libuksched] uk_thread 0xffffffc000ff8018 (xenstore): ctx:0xffffffc000ff8018, ectx:0xffffffc000ff9160, tlsp:0xffffffc000ff9020
dbg:  [libcontext] tls_area_init: target: 0xffffffc000ff9020 (312 bytes)
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: tcb: 16 bytes
dbg:  [libcontext] tls_area_init: pad: 0 bytes
dbg:  [libcontext] tls_area_init: copy (.tdata): 0 bytes
dbg:  [libcontext] tls_area_init: uninitialized (.tbss): 0 bytes
dbg:  [libcontext] (tls_area): ffffffc000ff9020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
dbg:  [libcontext] *
dbg:  [libcontext] ukarch_ctx 0xffffffc000ff8018: entry:0xffffffc00000ae14(0), sp:0xffffffc0000f0020
dbg:  [libukbus] Probe bus 0xffffffc0000a1178...
dbg:  [libxenbus] Complete main loop of xs_msg_write.
dbg:  [libxengic] EL1 IRQ#31 caught
dbg:  [libxengic] EL1 IRQ#1023 caught
dbg:  [libxenbus] Rsp_cons 0, rsp_prod 28.
dbg:  [libxenbus] Msg len 28, 28 avail, id 0.
dbg:  [libxenbus] Message is good.
dbg:  [libxenbus] Rsp_cons 28, rsp_prod 28.
dbg:  [libxenbus] Complete main loop of xs_msg_write.
dbg:  [libxengic] EL1 IRQ#31 caught
dbg:  [libxengic] EL1 IRQ#1023 caught
dbg:  [libxenbus] Rsp_cons 28, rsp_prod 44.
dbg:  [libxenbus] Msg len 16, 16 avail, id 1.
dbg:  [libxenbus] Message is good.
dbg:  [libxenbus] Rsp_cons 44, rsp_prod 44.
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
             Epimetheus 0.12.0~d25310a4
Hello world!
Arguments:  "helloworld"


    c'_'o  .--'
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'o'o  .--.        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .-.         RQ#1023 caught
    (| |)_/   `        dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'o'o  .--.        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .--'        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .--.        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .-.         RQ#1023 caught
    (| |)_/   `        dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .--.        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c-_-o  .--'        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
    c'_'o  .--.        RQ#1023 caught
    (| |)_/            dbg:  [libposix_time] (int) uk_syscall_r_nanosleep((const struct timespec*) 0xffffffc0000a0ec8, (struct timespec*) 0xffffffc0000a0ec8)
      _                RQ#27 caught
Detached from console
```

Destroy domain - DomU Unikraft will be destroyed and Xen domain id "4"
will not be accessible any more.

```
uart:~$ xu destroy 4
domain:4 destroyed
```

## Start Lunux linux_pv_domu domain example

TODO: cmds and logs

## Bootlog

The boot log will look like below when RPI5 is booted:

```
NOTICE:  BL31: v2.10.0(release):v2.10.0-958-g09a1cc2a0b-dirty
NOTICE:  BL31: Built : 14:54:20, May 10 2024
I/TC: 
I/TC: Non-secure external DT found
I/TC: pl011: device parameters ignored (115200n8)
E/TC:0 0   pl011_dev_init:216 pl011: unexpected register size: 200
I/TC: OP-TEE version: 4.2.0 (gcc version 13.2.0 (GCC)) #1 Fri Apr 12 09:51:21 UTC 2024 aarch64
I/TC: WARNING: This OP-TEE configuration might be insecure!
I/TC: WARNING: Please check https://optee.readthedocs.io/en/latest/architecture/porting_guidelines.html
I/TC: Primary CPU initializing
I/TC: Initializing virtualization support
I/TC: Primary CPU switching to normal world boot
I/TC: WARNING: Using fixed value for stack canary


U-Boot 2024.04 (May 31 2024 - 15:00:11 +0000)

DRAM:  1020 MiB (effective 8 GiB)
mbox: Header response code invalid
RPI 5 Model B (0xd04170)
Core:  44 devices, 15 uclasses, devicetree: board
MMC:   mmc@fff000: 0, mmc@1100000: 1
Loading Environment from FAT... Unable to read "uboot.env" from mmc0:1... 
In:    serial,usbkbd
Out:   serial,vidconsole
Err:   serial,vidconsole
mbox: Header response code invalid
bcm2835: Could not query MAC address
PCIe BRCM: link up, 5.0 Gbps x4 (!SSC)
Net:   eth0: ethernet@100000

starting USB...
No working controllers found
Hit any key to stop autoboot:  0 
No EFI system partition
No EFI system partition
Failed to persist EFI variables
No EFI system partition
Failed to persist EFI variables
No EFI system partition
Failed to persist EFI variables
** Booting bootflow '<NULL>' with efi_mgr
Loading Boot0000 'mmc 0' failed
EFI boot manager: Cannot load any image
Boot failed (err=-14)
** Booting bootflow 'mmc@fff000.bootdev.part_1' with script
Working FDT set to 2efec900
1179656 bytes read in 52 ms (21.6 MiB/s)
1994 bytes read in 2 ms (973.6 KiB/s)
341 bytes read in 2 ms (166 KiB/s)
11982 bytes read in 2 ms (5.7 MiB/s)
2961412 bytes read in 125 ms (22.6 MiB/s)
10149062 bytes read in 424 ms (22.8 MiB/s)
6116 bytes read in 2 ms (2.9 MiB/s)
Working FDT set to 1c00000
920 bytes read in 1 ms (898.4 KiB/s)
## Flattened Device Tree blob at 2efec900
   Booting using the fdt blob at 0x2efec900
Working FDT set to 2efec900
   Using Device Tree in place at 000000002efec900, end 000000002f005fff
Working FDT set to 2efec900

Starting kernel ...

 Xen 4.19-unstable
(XEN) Xen version 4.19-unstable (xtrs@) (aarch64-poky-linux-gcc (GCC) 13.2.0) debug=y 2024-02-02
(XEN) Latest ChangeSet: Fri Jan 5 10:46:44 2024 +0200 git:8ff1f6708a-dirty
(XEN) build-id: 9987ca876899bfc009d6377fd79ced2324a56e1b
(XEN) Processor: 00000000414fd0b1: "ARM Limited", variant: 0x4, part 0xd0b,rev 0x1
(XEN) 64-bit Execution:
(XEN)   Processor Features: 1100000010111112 0000000000000010
(XEN)     Exception Levels: EL3:64 EL2:64 EL1:64 EL0:64+32
(XEN)     Extensions: FloatingPoint AdvancedSIMD
(XEN)   Debug Features: 0000000010305408 0000000000000000
(XEN)   Auxiliary Features: 0000000000000000 0000000000000000
(XEN)   Memory Model Features: 0000000000101122 0000000010212122
(XEN)   ISA Features:  0000100010211120 0000000000100001
(XEN) 32-bit Execution:
(XEN)   Processor Features: 0000000010010131:0000000000010000
(XEN)     Instruction Sets: AArch32 A32 Thumb Thumb-2 Jazelle
(XEN)     Extensions: GenericTimer
(XEN)   Debug Features: 0000000004010088
(XEN)   Auxiliary Features: 0000000000000000
(XEN)   Memory Model Features: 0000000010201105 0000000040000000
(XEN)                          0000000001260000 0000000002122211
(XEN)   ISA Features: 0000000002101110 0000000013112111 0000000021232042
(XEN)                 0000000001112131 0000000000010142 0000000001011121
(XEN) Using SMC Calling Convention v1.5
(XEN) Using PSCI v1.1
(XEN) SMP: Allowing 4 CPUs
(XEN) Generic Timer IRQ: phys=30 hyp=26 virt=27 Freq: 54000 KHz
(XEN) GICv2 initialization:
(XEN)         gic_dist_addr=000000107fff9000
(XEN)         gic_cpu_addr=000000107fffa000
(XEN)         gic_hyp_addr=000000107fffc000
(XEN)         gic_vcpu_addr=000000107fffe000
(XEN)         gic_maintenance_irq=25
(XEN) GICv2: 320 lines, 4 cpus, secure (IID 0200143b).
(XEN) XSM Framework v1.0.1 initialized
(XEN) Flask: 128 avtab hash slots, 361 rules.
(XEN) Flask: 128 avtab hash slots, 361 rules.
(XEN) Flask:  4 users, 3 roles, 42 types, 2 bools
(XEN) Flask:  13 classes, 361 rules
(XEN) Flask:  Starting in permissive mode.
(XEN) Initialized GSX IRQ
(XEN) Using scheduler: SMP Credit Scheduler rev2 (credit2)
(XEN) Initializing Credit2 scheduler
(XEN)  load_precision_shift: 18
(XEN)  load_window_shift: 30
(XEN)  underload_balance_tolerance: 0
(XEN)  overload_balance_tolerance: -3
(XEN)  runqueues arrangement: socket
(XEN)  cap enforcement granularity: 10ms
(XEN) load tracking window length 1073741824 ns
(XEN) Allocated console ring of 32 KiB.
(XEN) CPU0: Guest atomics will try 17 times before pausing the domain
(XEN) Bringing up CPU1
I/TC: Secondary CPU 1 initializing
I/TC: Secondary CPU 1 switching to normal world boot
(XEN) CPU1: Guest atomics will try 15 times before pausing the domain
(XEN) CPU 1 booted.
(XEN) Bringing up CPU2
I/TC: Secondary CPU 2 initializing
I/TC: Secondary CPU 2 switching to normal world boot
(XEN) CPU2: Guest atomics will try 6 times before pausing the domain
(XEN) CPU 2 booted.
(XEN) Bringing up CPU3
I/TC: Secondary CPU 3 initializing
I/TC: Secondary CPU 3 switching to normal world boot
(XEN) CPU3: Guest atomics will try 2 times before pausing the domain
(XEN) Brought up 4 CPUs
(XEN) CPU 3 booted.
(XEN) I/O virtualisation disabled
(XEN) P2M: 40-bit IPA with 40-bit PA and 16-bit VMID
(XEN) P2M: 3 levels with order-1 root, VTCR 0x00000000800a3558
(XEN) Scheduling granularity: cpu, 1 CPU per sched-resource
(XEN) Initializing Credit2 scheduler
(XEN)  load_precision_shift: 18
(XEN)  load_window_shift: 30
(XEN)  underload_balance_tolerance: 0
(XEN)  overload_balance_tolerance: -3
(XEN)  runqueues arrangement: socket
(XEN)  cap enforcement granularity: 10ms
(XEN) load tracking window length 1073741824 ns
(XEN) Adding cpu 0 to runqueue 0
(XEN)  First cpu on runqueue, activating
(XEN) Adding cpu 1 to runqueue 0
(XEN) Adding cpu 2 to runqueue 0
(XEN) Adding cpu 3 to runqueue 0
(XEN) OP-TEE supports 4 simultaneous threads per guest.
I/TC: Removing guest 0
E/TC:0 0   virt_guest_destroyed:393 Client with id 0 is not found
(XEN) Using TEE mediator for OP-TEE
(XEN) alternatives: Patching with alt table 00000a00002f0d88 -> 00000a00002f1fa0
(XEN) CPU0 will use 24 loops workaround on exception entry
(XEN) CPU2 will use 24 loops workaround on exception entry
(XEN) CPU3 will use 24 loops workaround on exception entry
(XEN) CPU1 will use 24 loops workaround on exception entry
I/TC: Added guest 1
(XEN) *** LOADING DOMAIN 0 ***
(XEN) Loading d0 kernel from boot module @ 0000000000e00000
(XEN) Allocating 1:1 mappings totalling 128MB for dom0:
(XEN) BANK[0] 0x00000010000000-0x00000018000000 (128MB)
(XEN) Grant table range: 0x00000002000000-0x00000002040000
(XEN) Allocating PPI 16 for event channel interrupt
(XEN) d0: extended region 0: 0x2200000->0x7e00000
(XEN) d0: extended region 1: 0x8400000->0xfe00000
(XEN) d0: extended region 2: 0x18000000->0x1ce00000
(XEN) d0: extended region 3: 0x1f000000->0x3fa00000
(XEN) d0: extended region 4: 0x40000000->0x4fe00000
(XEN) d0: extended region 5: 0x70000000->0x1ffe00000
(XEN) Loading zImage from 0000000000e00000 to 0000000010000000-00000000102d3004
(XEN) Loading d0 DTB to 0x0000000017e00000-0x0000000017e13436
(XEN) avc:  denied  { create } for current=d[IDLE] scontext=system_u:system_r:xenboot_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) *** LOADING DOMU cpus=1 memory=0x80000KB ***
(XEN) Loading d1 kernel from boot module @ 0000000001200000
(XEN) d1: STATIC BANK[0] 0x00000050000000-0x00000070000000
I/TC: Added guest 2
(XEN) d1: extended region 0: 0x200000->0x7e00000
(XEN) d1: extended region 1: 0x8400000->0x1ce00000
(XEN) d1: extended region 2: 0x1f000000->0x37e00000
(XEN) d1: extended region 3: 0x39000000->0x3fa00000
(XEN) d1: extended region 4: 0x40000000->0x4fe00000
(XEN) d1: extended region 5: 0x70000000->0x1ffe00000
(XEN) Loading zImage from 0000000034000000 to 0000000050000000-0000000051aaba00
(XEN) Loading d1 DTB to 0x0000000058000000-0x0000000058001984
(XEN) avc:  denied  { create } for current=d[IDLE] scontext=system_u:system_r:xenboot_t tcontext=system_u:object_r:dom0_t tclass=event
(XEN) avc:  denied  { bind } for current=d[IDLE] scontext=system_u:object_r:dom0_t tcontext=system_u:system_r:dom0_t tclass=event
(XEN) Initial low memory virq threshold set at 0x4000 pages.
(XEN) Scrubbing Free RAM in background
(XEN) Std. Loglevel: All
(XEN) Guest Loglevel: All
(XEN) ***************************************************
(XEN) WARNING: SILO mode is not enabled.
(XEN) It has implications on the security of the system,
(XEN) unless the communications have been forbidden between
(XEN) untrusted domains.
(XEN) ***************************************************
(XEN) 3... 2... 1... 
(XEN) *** Serial input to DOM0 (type 'CTRL-a' three times to switch input)
(XEN) Freed 396kB init memory.
(XEN) d1v0 Unhandled SMC/HVC: 0x84000050
(XEN) d1v0 Unhandled SMC/HVC: 0x8600ff01
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER4
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER8
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER12
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER16
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER20
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER24
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER28
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER32
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER36
(XEN) d1v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER0
(XEN) avc:  denied  { getparam } for source=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:unlabeled_t tclass=hvm
(XEN) avc:  denied  { physmap } for source=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:unlabeled_t tclass=mmu
(XEN) avc:  denied  { query } for source=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:unlabeled_t tclass=grant
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER4
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER8
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER12
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER16
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER20
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER24
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER28
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER32
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER36
(XEN) d0v0: vGICD: unhandled word write 0x000000ffffffff to ICACTIVER0
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 1 to 2 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 2 to 3 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 3 to 4 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 4 to 5 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 5 to 6 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 6 to 7 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 7 to 8 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 8 to 9 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 9 to 10 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 10 to 11 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 11 to 12 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 12 to 13 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 13 to 14 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 14 to 15 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 15 to 16 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 16 to 17 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 17 to 18 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 18 to 19 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant It/TC: Reserved shared memory is enabled
abI/TC: Dynamic shared memory is enabled
leI/TC: Normal World virtualization support is enabled
 If/TC: Asynchronous notifications are disabled
rom 19 to 20 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 20 to 21 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 21 to 22 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 22 to 23 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 23 to 24 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 24 to 25 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 25 to 26 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 26 to 27 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 27 to 28 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 28 to 29 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 29 to 30 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 30 to 31 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 31 to 32 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 32 to 33 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 33 to 34 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 34 to 35 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 35 to 36 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 36 to 37 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 37 to 38 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 38 to 39 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 39 to 40 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 40 to 41 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 41 to 42 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 42 to 43 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 43 to 44 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 44 to 45 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 45 to 46 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 46 to 47 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 47 to 48 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 48 to 49 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 49 to 50 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 50 to 51 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 51 to 52 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 52 to 53 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 53 to 54 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 54 to 55 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 55 to 56 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 56 to 57 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 57 to 58 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 58 to 59 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 59 to 60 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 60 to 61 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 61 to 62 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 62 to 63 frames
(XEN) xen-source/xen/common/grant_table.c:1909:d0v0 Expanding d0 grant table from 63 to 64 frames
I: sdhc: detected ver:2 vendor_ver:16 supported
I: sdhc: capabilities caps0:15eac832 caps1:8000a577
I: mmc@1000fff000:"reset max bus frequency to 200000000 from 0"


** Booting Zephyr OS build cb85dfaa5099 ***
(XEN) avc:  denied  { getdomaininfo } for source=d0 target=d1 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=domain
(XEN) avc:  denied  { getparam } for source=d0 target=d1 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=hvm
(XEN) avc:  denied  { map_read map_write } for source=d0 target=d1 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=mmu
(XEN) avc:  denied  { create } for source=d0 target=d1 scontext=system_u:system_r:dom0_t tcontext=system_u:object_r:unlabeled_t tclass=event
(XEN) avc:  denied  { bind } for source=d0 target=d1 scontext=system_u:object_r:unlabeled_t tcontext=system_u:system_r:unlabeled_t tclass=event
(XEN) avc:  denied  { setparam } for source=d0 target=d1 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:unlabeled_t tclass=hvm
(XEN) avc:  denied  { send } for current=d0 scontext=system_u:system_r:dom0_t tcontext=system_u:object_r:unlabeled_t tclass=event
I: d(XEN) avc:  denied  { send } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:object_r:dom0_t tclass=event
om0less: attached 1 domains
32muart:~$ I: Card switched to 1.8V signaling
I: storage: block count 124735488, Sector size 512, Memory Size(MB) 60906
I: storage: mounted, binaries folder /0:/dom0/

uart:~$ 
uart:~$ (XEN) avc:  denied  { xen_extraversion } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_compile_info } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_capabilities } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_changeset } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_pagesize } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_commandline } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { xen_build_id } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=version
(XEN) avc:  denied  { physinfo } for current=d1 scontext=system_u:system_r:unlabeled_t tcontext=system_u:system_r:xen_t tclass=xen
fs ls /0(XEN) DOM1: 
(XEN) DOM1: Poky (Yocto Project Reference Distro) 5.0.1 raspberrypi5-domd ttyAMA0
(XEN) DOM1: 
:/dom0
z_sync.bin
z_blinky.bin
helloworld_xen-arm64
liunx_pv_image
uart:~$
```

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
