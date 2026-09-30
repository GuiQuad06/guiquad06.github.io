---
layout: default
title: Buildroot
---

# Buildroot

- [Resources](#resources)
- [Useful Commands](#useful-commands)
- [The Artefact Tree](#the-artefact-tree)
- [Concrete Example: Cross-Compiling a Qt Application](#concrete-example-cross-compiling-a-qt-application)

---

## Resources

- [Official website](https://buildroot.org)
- [User manual](https://buildroot.org/downloads/manual/manual.html)
- [GitHub mirror](https://github.com/buildroot/buildroot) (the canonical repo lives
  on `git.buildroot.org`; GitHub is a read-only mirror)
- [elinux.org Buildroot wiki](https://elinux.org/Buildroot) — boards, tips, FAQ

## Useful Commands

### Set a default configuration

```bash
make list-defconfigs          # list all available board defconfigs
make raspberrypi3_defconfig    # seed .config from a known board
```

### Tweak the configuration

```bash
make menuconfig   # ncurses UI
make xconfig      # Qt UI
make nconfig      # alternative ncurses UI with search
```

- `make savedefconfig` writes the minimal diff back to a `defconfig` file so it
  can be versioned (the full `.config` is not meant to be committed).

### Build

```bash
make            # build everything (toolchain, packages, filesystem, images)
make -j$(nproc) # parallel build
make clean      # remove build output, keep the toolchain/config
make distclean  # remove everything, including downloads and the toolchain
```

## The Artefact Tree

Buildroot does all its work under `output/`:

| Directory | Content |
|---|---|
| `output/build/` | Per-package extracted sources and intermediate build directories. |
| `output/host/` | The cross-toolchain and host-side tools (compiler, `qmake`, etc.) — built to run **on the build machine**, used to build everything else, never shipped to the target. |
| `output/staging/` | A sysroot mirroring the target filesystem but keeping headers, static libs and `.la`/`.pc` files — used when cross-compiling other packages or external applications against target libraries. |
| `output/target/` | The staged root filesystem as it will run on the target (stripped binaries, no headers/static libs) — this is what gets packaged into the final image. |
| `output/images/` | The final binary artefacts ready to flash: kernel image, DTBs, bootloader, and the root filesystem image(s) (`.ext2/4`, `.squashfs`, `.tar`, etc.). |

## Concrete Example: Cross-Compiling a Qt Application

Once Buildroot has built its Qt5 packages, `output/host/` contains a target-aware
`qmake` plus the matching `mkspec`. Point your project at them instead of using
the host's own Qt install:

```bash
~/buildroot/output/host/usr/bin/qmake ../patisserie_cute/patisserie_cute.pro \
  -spec ~/buildroot/output/host/mkspecs/linux-aarch64-gnu-g++ CONFIG+=debug
```

```bash
make
```

- `output/host/usr/bin/qmake` is the Buildroot-built `qmake` — it knows to generate
  a `Makefile` that invokes the cross-toolchain (also under `output/host/`) rather
  than the native one.
- `-spec linux-aarch64-gnu-g++` selects the mkspec matching the target
  architecture/toolchain (here AArch64 with the GNU toolchain).
- The resulting binary links against the libraries staged in `output/staging/`
  and is meant to run on the target, not the build host.

[Back to Embedded Linux](./)
