# MT6761 Kernel 4.9.117 (Android)

Huawei Y5 2019 Downstream Android kernel.

Linux **4.9.117** for the **MT6761** (Helio A22) platform.

Defconfig: `merge_full_k61v1_64_mex_a32_defconfig`. Built and verified with the `aarch64-linux-android-4.9` (GCC 4.9) cross-toolchain, configured for an ARM64 device tree blob in the final image.

## Requirements

- Linux (tested on Arch / CachyOS)
- `make`, `git`, `bc`, `dtc`
- Python 3 (the bundled DCT code generator is Python 3 only)
- AArch64 cross-toolchain **aarch64-linux-android-4.9** (GCC 4.9)

> **Important:** this tree must be compiled with the matching **GCC 4.9** toolchain.
> Newer compilers fail against the legacy Android/MediaTek 4.9-era code
> (old flags, `-Werror`-style warnings and pre-warning errors).

A prebuilt toolchain can be fetched from:
[Adrilaw/aarch64-linux-android-4.9-toolchain](https://github.com/Adrilaw/aarch64-linux-android-4.9-toolchain)

On Arch-based systems install the remaining build deps with:

```sh
sudo pacman -S --noconfirm bc dtc
```

## Build

Point `CROSS_COMPILE` at your toolchain, then configure and build.

```sh
export PATH=$PATH:/path/to/aarch64-linux-android-4.9/bin
export CROSS_COMPILE=aarch64-linux-android-

make ARCH=arm64 merge_full_k61v1_64_mex_a32_defconfig
make ARCH=arm64 -j$(nproc) Image.gz-dtb
```

If you edit the top-level `Makefile` instead, set:

```make
CROSS_COMPILE = /path/to/aarch64-linux-android-4.9/bin/aarch64-linux-android-
```

### Output

| Artifact                    | Path                                |
|-----------------------------|-------------------------------------|
| Kernel image + DTB          | `arch/arm64/boot/Image.gz-dtb`      |
| Plain kernel image          | `arch/arm64/boot/Image`             |

### Clean

```sh
make ARCH=arm64 distclean
```

## Patches in this tree

- **Build fixes (top-level `Makefile`)**
  - `CROSS_COMPILE` for the 4.9 Android toolchain
  - `-fcommon` added to `HOSTCFLAGS` so the bundled `dtc` builds under modern host GCC (fixes `yylloc` redefinition)
  - `export MTK_PLATFORM := $(subst ",,$(CONFIG_MTK_PLATFORM))` and global `KBUILD_CFLAGS -I` for the MediaTek `mt-plat` headers
- **Python 3 port** — DCT code generator (`tools/dct`) ported from Python 2 to Python 3
- **Include path fixes** — ~50 `-I` flags added across MediaTek sub-Makefiles to resolve missing local headers
- **Platform variable hygiene** — `$(CONFIG_MTK_PLATFORM)` (value carries quotes) replaced with clean `$(MTK_PLATFORM)` in Makefiles
- **Connectivity** — Makefile now tolerates a missing vendor connectivity tree and falls back to in-tree drivers; a weak stub keeps `accdet` linked

## Known limitations

- The `vendor/mediatek/kernel_modules/connectivity` module tree (WMT / WLAN / BT / GPS / FM)
  is **not part of this repository**. Those drivers are skipped on purpose, so
  Wi-Fi / Bluetooth / GPS / FM radio functionality is **not built**.
  The `setHeadsetStatus` symbol, normally exported by the FM driver, is provided
  as a weak stub in `drivers/misc/mediatek/accdet/mt6357/accdet.c`.
- Requires the GCC 4.9 toolchain (see above).

## Repository structure

```
kernel/
├── arch/                 # arm64 support, defconfigs, boot output
├── drivers/              # device drivers (incl. MediaTek platform drivers)
├── tools/dct/            # DCT (device config) code generator (Python 3)
├── Makefile              # build entry point (CROSS_COMPILE, platform vars)
└── README.md
```

The Git history contains the build fixes described above; the untouched 2019
upstream source lives in the working tree of this repository.

## License

This is the Linux kernel tree. The kernel is licensed under GPL-2.0 — see
`COPYING` for details.
