---
layout: default
title: OpenOCD
---

# OpenOCD

**OpenOCD** (Open On-Chip Debugger) is a daemon that talks to a debug probe
(ST-Link, J-Link, FTDI, ...) over JTAG/SWD on one side, and exposes a
**GDB server** (default TCP port `3333`) plus a **telnet console** (default
port `4444`) on the other side. GDB never talks to the hardware directly —
it connects to OpenOCD, which does the actual flashing/halting/stepping.

```
gdb-multiarch  --remote :3333-->  OpenOCD  --JTAG/SWD-->  Target MCU
                                     ^
                                     |
                              telnet :4444
```

## 1. Install

```bash
sudo apt install openocd gdb-multiarch
```

## 2. Start the OpenOCD server

Pick an interface config (matches your probe) and a target config (matches
your MCU/board), then start the daemon:

```bash
openocd -f interface/stlink.cfg -f target/stm32f3x.cfg
```

- Board-specific `.cfg` files (under `board/`) often combine both in one file.
- Leave this process running in its own terminal — it stays attached to the
  probe and prints GDB/telnet connection info once the target is detected.

## 3. Connect with GDB

```bash
gdb-multiarch build/firmware.elf
```

- `gdb-multiarch` is a build of GDB that supports many target architectures
  (ARM, RISC-V, MIPS, ...) from a single binary, instead of needing an
  architecture-specific GDB (e.g. `arm-none-eabi-gdb`).

Inside the GDB prompt:

```
(gdb) target extended-remote localhost:3333
```

- `target extended-remote` attaches GDB to the OpenOCD GDB server. The
  **extended** variant (vs plain `remote`) additionally allows GDB to
  restart/kill the program on the target, not just debug an already-running
  one.

Then, typically:

```
(gdb) monitor reset halt   # ask OpenOCD to reset the MCU and halt it
(gdb) load                 # flash the .elf onto the target
(gdb) break main
(gdb) continue
```

## 4. TUI mode

```
(gdb) lay next
```

- `lay` is short for `layout`. `layout next` cycles through GDB's Text User
  Interface layouts (source, assembly, split source+asm, registers), which
  is handy for stepping through code with live register/source views instead
  of the plain command-line prompt.
- Other useful TUI shortcuts: `layout src`, `layout asm`, `layout regs`,
  `Ctrl-x a` (toggle TUI on/off), `Ctrl-x o` (switch active window).

## 5. Useful telnet commands (optional, separate session)

```bash
telnet localhost 4444
```

```
> reset halt
> mdw 0x20000000 4     # memory display, word, 4 words
> flash write_image erase build/firmware.elf
```

[Back to Embedded Systems](./)
