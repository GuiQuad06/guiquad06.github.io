---
layout: default
title: Yocto Project
---

# Yocto Project

- [General Notes](#general-notes)
- [Tutorial](#tutorial)
- [Miscellaneous Notes](#miscellaneous-notes)

---

## General Notes

[The Yocto Project](https://www.yoctoproject.org/)

- At the start, source the build environment: `source oe-init-build-env <build-dir>`.
- 2 important commands:
  - `bitbake-layers create-layer ../meta-toto`
  - `bitbake-layers add-layer ../meta-toto`
- 2 main images:
  - `bitbake core-image-minimal` (the strict minimum to boot)
  - `bitbake core-image-base` (minimal + peripherals)

## Tutorial

### Yocto Project #1: Base Image

- Look up the desired Yocto version.
- `git clone` into a directory and install the required tools.
- `cd poky`
- `source oe-init-build-env`
- `build/conf/bblayers.conf` contains the meta-layers to build into the kernel.
- In `local.conf`, define the target (`qemux86` for the emulator).
- `bitbake core-image-base` (build).
- Image available in `tmp/deploy/qemux86-64` (emulator).
- From `build`, run:
  - `runqemu qemux86-64`

### Yocto Project #2: Base Image with BSP for the iMX233 Eval Board

- `-b kirkstone` — lets you specify the poky distro when cloning a BSP meta-layer.
- `source oe-init-build-env`
- Add the BSP meta-layers to `bblayers.conf` in `build/conf`.
- Choose the machine in `local.conf` under `build/conf` (check the manufacturer's
  meta-layer for the SoC, or the 3rd-party meta-layer for the eval board!).
- Add `ACCEPT_FSL_EULA = "1"` to `local.conf`.
- `bitbake core-image-base`
- Look in `build/tmp/deploy/image...` for the generated files:
  - `xxx.dtb` (device tree blob)
  - `uboot.sb` (U-Boot)
  - `uImage` (kernel image)
- Extract `xxx.rootfs.wic.gz`, then `dd` it.
- On Windows, flash directly from the `wic.gz` with Balena Etcher.
- Burn the SD card (Balena Etcher with the extracted `wic`, or always with `dd` on
  the `wic`).
- `uImage` and the device tree go in the boot partition, the rootfs in root,
  U-Boot in partition 1.

### Yocto Project #3: Linux Boot Process

- N/A

### Yocto Project #4: Linux Driver

- Launch VS Code with the C/C++ extension.
- Create a simple template with a kernel-object Makefile.
- `make`
- `lsmod`
- `sudo insmod kernel_driver.ko`
- Check `dmesg | tail`
- `sudo rmmod` to remove the module.
- Copy a `recipes-kernel` template from `meta-skeleton` into the 3rd-party
  meta-layer.
- `LICENSE = "CLOSED"` in the `*.bb`.
- Add `SRC_URI` = Makefile + C file, and `RPROVIDES` = kernel object name without
  the `.ko`.
- Keep the Makefile proposed by the skeleton.
- Source the build env, then `bitbake test_recipe`.
- Add `IMAGE_INSTALL_append += "test_recipe"` and
  `KERNEL_MODULE_AUTOLOAD += "test_driver"` to `local.conf`.
- `bitbake core-image-base`
- Command to add the recipe meta-layer to `bblayers.conf` automatically:
  - `bitbake-layers create-layer ../meta-quad`
  - `bitbake-layers add-layer ../meta-quad` (updates `bblayers.conf`)
- Flash the SD card and test.

### Yocto Project #5: GUI Linux Image using Arago

- N/A

### Yocto Project #6: Embedded Linux Application with Yocto Project and Qt

Embedded Linux application with Yocto Project and Qt for a user application:

- Ubuntu machine.
- Create the workspace directory.
- Write a hello-world for the sake of it.
- Test the app standalone:

  ```bash
  gcc xxx.c -o xxx
  ./xxx
  ```

- `git clone` the Yocto Project.
- Create a meta-layer:

  ```bash
  ./oe-init-build-env
  bitbake-layers create-layer ../meta-quad
  bitbake-layers add-layer ../meta-quad
  cd conf
  ```

- Check `bblayers.conf`.
- `cd meta-quad`
- `mkdir recipes-quad`
- Write the app's `*.bb` (`LICENSE = "CLOSED"`).
- Copy the C program into the recipe.
- Add `SRC_URI` in the `*.bb`, and also `S = "${WORKDIR}"`.
- Add `do_compile` and `do_install`:

  ```bash
  do_compile{
      ${CC} myCapp.c -o myCapp
  }
  do_install{
      install -d ${D}${bindir}
      install -m 0775 myCapp ${D}${bindir}/
  }
  bitbake myCapp
  ```

- Add the recipe to the kernel image:
  - `IMAGE_INSTALL_append += "myCapp"` in the `local.conf` file.
- Build the image:
  - `bitbake tisdk-base-image`
  - Delete `build/tmp` if there are errors.

  ```bash
  unxz ./xxx.rootfs.wic.xz
  sudo dd if=./xxx.wic of=/dev/sdb bs=1M \
     iflag=fullblock oflag=direct conv=fsync status=progress
  ```

- Check the application in the `bin` directory on the SD card.
- Put it on the target, test.

- `touch myQT.cpp` then `touch myQT.pro`
- In the `.pro` file:
  - `TEMPLATE = app`
  - `TARGET = myQT`
  - `INCLUDEPATH += .`
  - `QT += gui widgets`
  - `DEFINES += QT_DEPRECATED_WARNINGS`
  - `SOURCE += myQT.cpp`
- `qmake` then `make`.
- Run it on the host machine to test.
- `meta-quad/recipes-software`, then add the source and project file.
- Add `DEPENDS += " qtbase wayland "` to the `*.bb`.
- Add `inherit qmake` to the `*.bb`.
- `FILES_${PN} += "${bindir}/myQT"`
- `bitbake tisdk-base-image`
- Flash and test.

## Miscellaneous Notes

- Kernel overlay (module, driver) goes in `recipes-kernel`.
- Image overlay goes in `meta/recipes-core`.
- App goes in `recipes-apps`.
- QEMU BSP goes in `meta-yocto-bsp`.
- Raspberry Pi BSP goes in `meta-raspberrypi`.

Load an overlay in `local.conf`:

```bash
RPI_EXTRA_CONFIG:append = "\nenable_uart=1\ndtoverlay=disable-bt\ndtoverlay=dht11,gpiopin=4\n"
RPI_KERNEL_DEVICETREE_OVERLAYS:append = " overlays/dht11.dtbo"
```

Typical layer tree:

```bash
meta-mystation/
├── conf/
│   └── layer.conf                 # priority, recipe patterns
├── recipes-apps/
│   └── sensord/
│       ├── sensord_1.0.bb
│       └── files/
│           ├── sensord.c
│           └── sensord.service     # (bonus systemd)
├── recipes-kernel/
│   └── linux/
│       ├── linux-raspberrypi_%.bbappend
│       └── files/
│           └── dht11.cfg           # kernel config fragment
└── recipes-core/
    └── images/
        └── mystation-image.bb      # final image
```

[Back to Embedded Linux](./)
