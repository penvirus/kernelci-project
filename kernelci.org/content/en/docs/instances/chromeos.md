---
title: "ChromeOS instance"
date: 2022-07-20T08:30:00Z
description: "How chromeos.kernelci.org works"
---

ChromeOS instance is co-hosted on [staging.kernelci.org](https://staging.kernelci.org)
instance server.
The main purpose of this instance is to test the functionality of new kernel versions 
with ChromeOS userspace, compiled on KernelCI servers from publicly available sources (ChromiumOS).

Since ChromeOS requires different way of kernel features configuration than regular distributions,
and rootfs builds also significantly different too, this instance uses a separate 
kernelci-core and kernelci-jenkins project branches chromeos/chromeos.kernel.org.

## How Chrome devices testing works, in general

It is assumed that the reader has some experience with [`LAVA`](https://www.lavasoftware.org/index.html).

In the traditional Chromebook boot process - bootloader looks for a 
special kernel partition, and then loads the latest working kernel 
[`(ChromeOS boot documentation)`](https://chromium.googlesource.com/chromiumos/docs/+/HEAD/disk_format.md#Google-ChromeOS-devices) . 
In our testing, we take control of the device before this stage and we 
load the tested kernel over the network, via 
[`LAVA depthcharge`](https://docs.lavasoftware.org/lava/actions-boot.html#depthcharge).

An additional complication is that we cannot place kernel modules on the nfs image 
or initrd like "usual" LAVA DUT, due to the ChromeOS architecture, so in the case of ChromeOS, 
we need to _preload_ tested kernel modules to device persistent storage 
partition 3 on the DUT(Chromebook).
This is done via 2-stage LAVA job, pay attention to this section test, 
namespace modules: [`cros-boot.jinja2`](https://github.com/kernelci/kernelci-core/blob/chromeos/config/lava/chromeos/cros-boot.jinja2).

It is worth noting that testing in QEMU is a bit different from testing 
on Chromebooks, so installing modules is done in the rootfs file using 
deploy postprocess:
[`cros-boot-qemu.jinja2`](https://github.com/kernelci/kernelci-core/blob/chromeos.kernelci.org/config/lava/chromeos/cros-boot-qemu.jinja2).

In the next step, we expect a login prompt on the serial console and
successfull login over it, but due to reliability problems for LAVA, 
we use serial console minimally. Success indicates kernel and operating system loaded
properly.

And the last step is to run the docker image in LAVA and connect to the DUT 
via ssh using the developer key and perform the necessary tests.


## Building and structure of ChromeOS rootfs

Building a ChromeOS rootfs image involves building by [`Google recipe`](https://chromium.googlesource.com/chromiumos/docs/+/main/developer_guide.md#Building-ChromiumOS)
with our own tweaks, as following:
* Enable USB serial support for console
* Build not only by a specific manifest tag, but fix (manifest snapshot)
to certain commits level, this will allow us to get identical builds 
and consistent test results on all devices 
* Add generated ".bin" image to special "flasher" debos image to save
time
* Extract the kernel and modules for -fixed tests from the generated image 
and publish them separately as bzImage and modules.tar.xz
* Extract required tast files to run tast tests in LAVA Docker (tast.tgz)

You can build it manually using following commands, for qemu (amd64-generic):
```
git clone -b chromeos https://github.com/kernelci/kernelci-core/
cd kernelci-core
mkdir temp
chmod 0777 temp
docker run \
    --name kernelci-build-cros \
    -itd \
    -v $(pwd):/kernelci-core \
    --device /dev/kvm \
    -v /dev:/dev \
    --privileged \
    kernelci/cros-sdk
docker exec \
    -it kernelci-build-cros \
    ./kci_rootfs \
    build \
    --rootfs-config chromiumos-amd64-generic \
    --data-path config/rootfs/chromiumos \
    --arch amd64 \
    --output temp
```
Or specify the appropriate parameters in jenkins chromeos/rootfs-builder, 
and after that the rootfs will be automatically published on [`storage server`](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/)
