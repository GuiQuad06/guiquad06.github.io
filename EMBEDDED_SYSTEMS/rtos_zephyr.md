---
layout: default
title: Zephyr
---

# Zephyr

- [Moving from FreeRTOS to Zephyr](#moving-from-freertos-to-zephyr)
- [Tracing with Trace Compass](#open-source-tool-for-tracing-trace-compass)
- [Environment & samples](#environment--samples)
- [RAM / ROM usage & static analysis](#ram--rom-usage--static-analysis)
- [Flashing & debugging](#flashing--debugging)
- [Sample apps](#sample-apps)
- [Kconfig TUI](#kconfig-tui)
- [Application development](#application-development)

---

## Moving from FreeRTOS to Zephyr

- Why: bigger open source community.
- Scalable, hardware agnostic.
- Uses an OSAL.
- Move to CMake.
- West for project configuration (manifest).
- Leverage the device tree.
- `k_msgq` for small fixed-size messages.
- `k_fifo` for a linked-list queue.
- Boosting workflow.
- Emulator: Wokwi or Renode.
- [Puncover](https://github.com/HBehrens/puncover) for memory footprint — analyses
  C/C++ build output for code size, static variables, and stack usage.
- Thread analyzer.
- Twister as a test framework.
- [example-application](https://github.com/zephyrproject-rtos/example-application) —
  example out-of-tree application that is also a module.
- [Additional Zephyr extension commands](https://docs.zephyrproject.org/latest/develop/west/zephyr-cmds.html#software-bill-of-materials-west-spdx)
  — West SPDX / software bill of materials.

## Open-source tool for tracing: Trace Compass

Trace Compass has been used on
[Fancy Theremin](https://github.com/GuiQuad06/fancy-theremin/tree/feature/tracing) to
showcase how to use tracing on a Zephyr RTOS.

The `feature/tracing` branch is ready to be traced with those mechanisms:

- `app/overlay-trace.conf` — Kconfig file to enable tracing modules
- `scripts/trace/dump_trace.gdb` — script to dump the RAM buffer to the file system

1. Build with tracing enabled:
   ```bash
   west build -b $BOARD app -- -DEXTRA_CONF_FILE=overlay-trace.conf
   ```
2. Flash and let it run for a few seconds so the buffer fills:
   ```bash
   west flash
   ```
3. Attach the debugger:
   ```bash
   west debug
   ```
4. Inside the GDB session, halt and dump:
   ```gdb
   source ../scripts/trace/dump_trace.gdb
   ```

---

## Environment & samples

**Activate the environment:**

```bash
source ../.venv/bin/activate
```

**Build a sample:**

```bash
west build -b pic32cm_pl10_cnano -p -s samples/basic/blinky -d build_blinky -- -DCONFIG_STACK_USAGE=y
```

**Hello world on QEMU:**

```bash
west build -b qemu_x86 samples/hello_world -d build_hello
west build -t run
```

---

## RAM / ROM usage & static analysis

```bash
west build -t ram_plot        # RAM usage
west build -t rom_plot        # ROM usage
west build -t puncover        # static analysis (Puncover)
```

---

## Flashing & debugging

```bash
west flash -d build_blinky
west debug -d build_blinky    # debugging with pyOCD
```

Build artefact located at `build/zephyr/zephyr.elf`.

---

## Sample apps

- **Basic synchro** — 2 threads with a semaphore for synchronization.
- **Philosophers** — preemptive / cooperative threads.
- **Basic thread** — 2 threads with a FIFO (queue) sending something to print from A
  to B; same priority, two different waits (`k_msleep`).

---

## Kconfig TUI

```bash
west build -t menuconfig
```

---

## Application development

- This is workspace application development.
- The git repo should sit at the same level as `modules/`, `zephyr/`, etc., e.g.
  `myFirstApp/`.
- All `west` commands should be run from inside `myFirstApp`:
  ```bash
  west build -b pic32cm_pl10_cnano -p -s app -t menuconfig
  ```
- `app/` is the relevant folder for the core application development (source,
  overlay, specific Kconfig, …).

[Back to Embedded Systems](./)
