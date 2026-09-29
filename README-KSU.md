# KernelSU-Next + SUSFS kernel for GrapheneOS (Pixel 6 / 6 Pro)

> **Unofficial.** Not affiliated with, endorsed by, or supported by GrapheneOS.
> Don't ask for help with this kernel in GrapheneOS channels.

A fork of [GrapheneOS/kernel_common-6.1](https://github.com/GrapheneOS/kernel_common-6.1)
with root, root hiding and extra features added. It keeps GrapheneOS's own
kernel source and **the phone's stock, GrapheneOS-signed vendor modules**,
so only `boot.img` changes.

| Branch | GrapheneOS kernel tag | Tested on |
|---|---|---|
| `ksu-2026091900` | `2026091900` (also used by `2026092500`) | Pixel 6 (oriole), GrapheneOS 2026091901 |

Pixel 6 Pro (raven) builds from the same source but is **untested**.

## Features

| Feature | Source |
|---|---|
| KernelSU-Next v3.4.0 | [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next) `d575262` (vendored in `drivers/kernelsu`) |
| SUSFS | [simonpunk/susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu) `gki-android14-6.1` + pershoot `-dev` fixes |
| Unicode bypass fix | [WildKernels/kernel_patches](https://github.com/WildKernels/kernel_patches) |
| NoMount | [maxsteeel/nomount](https://github.com/maxsteeel/nomount) `6b1be18` (vendored in `fs/nomount`) |
| Baseband Guard | [vc-teahouse/Baseband-guard](https://github.com/vc-teahouse/Baseband-guard) `a54e0dc` + defensive SID-lookup change (vendored in `security/baseband-guard`) |
| NTSync | WildKernels NTSync patches |
| DroidSpaces support | SYSVIPC, POSIX_MQUEUE, IPC/PID/USER namespaces, DEVTMPFS, BINFMT_MISC ([DroidSpaces-OSS](https://github.com/ravindu644/Droidspaces-OSS)) |
| Networking | BBR (default), CAKE, IPSet, netfilter extras, CIFS. WireGuard is already in GrapheneOS. |
| TMPFS / BPF | POSIX ACLs and xattrs; BTF, eBPF and FUSE-BPF are already in GrapheneOS |

Mountify and the NoMount metamodule are KernelSU modules; install them from the KernelSU-Next manager.

## ⚠️ Security trade-offs

This kernel **removes parts of GrapheneOS's hardening**:
- Forced module signature verification and kernel lockdown are disabled, which is
  required so the stock vendor modules load under a kernel with a different signing key.
- Root is added (KernelSU-Next), and user namespaces are enabled for root
  (unprivileged creation is still blocked).

Only use it if you understand and accept that.

## Compatibility: why each release matches exactly one GrapheneOS kernel

GrapheneOS builds with `CONFIG_RANDSTRUCT_FULL`. The randomization seed is
derived from the build timestamp, which is the **committer date of HEAD**.
The vendor modules on the phone only load if the seed matches. So:

- Every commit on these branches keeps the committer date of the GrapheneOS tag
  commit (`GIT_COMMITTER_DATE="1789418757 -0400"` here). **Don't rebase or
  amend without it**, or every vendor module will fail with a `version magic` error.
- After a GrapheneOS update, check whether its kernel tag still points to the
  same commit. If it doesn't, the kernel must be rebuilt on the new tag.
- `uname -r` ends in the fork's commit hash. That's harmless, because modversions ignore the release string.

## Building

Needs a Linux host with ~25 GB free (the tree plus build output is ~15 GB) (WSL2 works; keep the tree on the Linux filesystem, not `/mnt/c`).

```bash
git clone --branch 2026091900 --depth 1 --recurse-submodules --shallow-submodules \
    https://gitlab.com/grapheneos/kernel_pixel_6.1.git
cd kernel_pixel_6.1/common/ack
git remote add ksu https://github.com/raigon-pawa/kernel_common-6.1-ksu.git
git fetch ksu ksu-2026091900 && git checkout ksu-2026091900
cd ../..
KLEAF_REPO_MANIFEST=aosp_manifest.xml ./build_raviole.sh --lto=full
strings out/raviole/dist/Image | grep -m1 "Linux version 6"   # must end: Mon Sep 14 20:45:57 UTC 2026
```

## Installing

Inject the kernel into **your own stock** GrapheneOS `boot.img` (from the factory
image for the exact build you run), using `magiskboot`:

```bash
./magiskboot unpack boot.img
cp /path/to/Image kernel
./magiskboot repack boot.img boot_ksu.img
```
```sh
adb reboot bootloader
fastboot boot boot_ksu.img        # test first; a reboot returns to your current kernel
fastboot flash boot boot_ksu.img  # only after it boots fine
```
Keep the stock `boot.img` so you can flash it back.

## Licenses

The kernel is GPL-2.0. KernelSU-Next's kernel part and Baseband Guard are GPL-2.0.
**SUSFS and NoMount are published under GPL-3.0**, which is not compatible with the
GPL-2.0-only kernel for binary distribution. Their authors' intent (e.g.
`MODULE_LICENSE("GPL")`) is noted, but this has not been clarified with them.
Keep that in mind before redistributing binaries.

## Credits

GrapheneOS, KernelSU-Next, simonpunk & pershoot (SUSFS), maxsteeel (NoMount),
vc-teahouse (Baseband Guard), ravindu644 (DroidSpaces-OSS), WildKernels.
