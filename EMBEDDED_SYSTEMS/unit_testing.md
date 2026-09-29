---
layout: default
title: Unit Testing (CTest / Unity)
---

# Unit Testing (CTest / Unity)

- CTest + Unity STM example repo:
  [Elmot repo](https://github.com/elmot/ctest-unity-stm32-testing-example)
- Best framework for C: **Unity** with **CTest**
- Semi-hosting with SWD/JTAG, stdin/stdout on host
- File system for the test report
- `add_test` in CMake
- Strategy: choose a higher-memory target for the test framework overhead
- Consider running tests from RAM

## The three testing levels

1. **Development testing**
   - Algorithms & HMI
   - Workflow: CMake / (CTest → Unity)
   - Checked out at commit: *Simple CTest example-application*
   ```bash
   mkdir build
   cd build
   cmake ..
   cmake --build .
   ctest -C Debug
   ```
2. **Emulator testing**
   - Algorithms testing only
   - Install QEMU
   - Set up semi-hosting (compiler flags, header-only for `syscalls.c`/`sysmem.c`)
   ```bash
   cmake --preset arm-test --fresh
   cmake --build ./build-arm-test
   ctest --preset arm-test-qemu --extra-verbose --output-junit junit-log.xml  # emulation testing w/ QEMU
   ```
   - Runs on the host with a cross-compiled binary.
3. **On-target testing**
   - Real-world interactions & communications
   - Install OpenOCD
   - Create a similar project in STM32CubeIDE with that hardware: NUCLEO-F302R8
   ```bash
   ctest --preset arm-test-hw --extra-verbose --output-junit junit-log.xml
   ```
   - Flashes the test ELF through ST-Link v2, resets and runs the firmware.

Other angles worth remembering: HMI host-based testing, algorithm emulator testing,
and real-world interaction/communication testing on the device.

---

## Unity framework quickstart

**Compile a test binary directly against the Unity sources:**

```bash
gcc TestDumbExample.c DumbExample.c ../unity/src/unity.c -I ../unity/src -o TestDumbExample
```

**Auto-generate the test runner file with the Ruby script** (`sudo apt install ruby`):

```bash
./../unity/auto/generate_test_runner.rb DumbExample.c TestDumbExample.c
```

[Back to Embedded Systems](./)
