---
title: "ChromeOS image changelog"
date: 2023-02-03
weight: 4
---

This file tracks divergences between the KernelCI built ChromiumOS images and upstream ChromiumOS as well as any changes between image relases.

Divergences can be considered tech-debt and in the long run need to be kept under control and minimized, therefore this chanelog should reflect its evolution from version to version.

### Fetching latest images

It is recommended to use the latest published image versions for each board from [this list](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/).

The latest version can be found either from the directory date name (e.g. `chromiumos-asurada/20230208.0`) or by the `distro_version` field in the `manifest.json` file, where for e.g. R106 is greater than R100.

## Release 106

### Supported boards

Direct links for each supported board in this release are provided here for convenience:
- [amd64-generic](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-amd64-generic/20221102.0/arm64)
- [asurada](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-asurada/20230208.0/arm64)
- [coral](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-coral/20221026.0/amd64)
- [dedede](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-dedede/20221113.0/amd64/)
- [grunt](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-grunt/20221028.0/amd64/)
- [hatch](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-hatch/20221027.0/amd64/)
- [jacuzzi](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-jacuzzi/20230206.0/arm64/)
- [nami](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-nami/20221120.0/amd64/)
- [octopus](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-octopus/20221025.0/amd64/)
- [rammus](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-rammus/20221116.0/amd64/)
- [sarien](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-sarien/20221111.0/amd64/)
- [trogdor](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-trogdor/20230214.0/arm64/)
- [volteer](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-volteer/20221115.0/amd64/)
- [zork](https://storage.chromeos.kernelci.org/images/rootfs/chromeos/chromiumos-zork/20221115.0/amd64/)

### Changes since previous version (R100)
	- A custom repo manifest is used to build images which points to forked repositories.
	- SElinux is now disabled in userspace, see issue https://github.com/kernelci/kernelci-core/issues/1372 .
	- chromeos-kernel-upstream is used for Mediatek image builds.

### Known divergences

#### src/third-party/kernel/upstream
	- Based on latest v6.2 kernel release.
	- `arch/arm64/configs/defconfig` was extended with Mediatek specific config fragments. In the future we might find a better way to fetch these for the upstream kernel builds.
	- Backported and cherry-picked ~ 19 patches to enable Panfrost on mediatek. These will be dropped in future kernel versions.

#### src/third-party/chromiumos-overlay
	- Disable selinux in the global profile for all boards.
	- Upgrade mesa-panfrost to latest 22.3.3 for Mali valhall GPU support.
	- Add USE flag to skip cr50 FW upgrades.
	- Bump ebuilds for divergence in other components (kernel, minigbm).

#### src/platform/minigbm
	- Add patch to allow minigbm to work with panfrost BO ioctls. This works but needs significant changes before being sent upstream.

#### src/overlays
	- Added fix for broken upstream chipset-mt8183 virtual/opengles panfrost dependencies.
	- Panfrost added as a graphics alternative for all Mediatek chipsets.
	- Removed Mali G-57 empty job workaround firmware which is not required for upstream graphics.
	- Instructed mt8183/8192 builds to use upstream kernel.
