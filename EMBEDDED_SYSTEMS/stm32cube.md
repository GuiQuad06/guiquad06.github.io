# STM32 Cube Cheatsheet

## STM32 Cube IDE

### Importing projects

1. Import project in the project explorer
2. General section -> Existing project into workspace (folder shall contain .cproject && .project files)

### Creating projects

Muliple ways to create a new project:
1. New STM32 Cube project
    - Choose the board or the example on the wizard
    - Poject name,
    - Language (C/C++),
    - Exe or static lib,
    - STM32 Cube project (good for Quick PoC), Empty project for : baremetal project (no-IOC / no-HAL)

2. New STM32 Cube project from an existing CubeMX config file (.ioc)
    - Same as previous one but by importing already done .ioc file

3. New STM32 CMake Project
    - Project from CMake template (exe or static lib as templates)

## STM32CubeIDE VScode extension (seamless, much better than Eclipse-based Bazaar)

- [User Manual](https://dev.st.com/stm32cube-docs/stm32cubeide-vscode/latest/en/docs/markup/getting_started/first_project_creation.html#first-project-creation)

### Creating projects

1. Launch STM32CubeMX (good for quick projects, PoC...)
    - Select appropriate board
    - Configure peripherals / interfaces
    - Add SW packages
    - Configure Clock tree
    - Project config (build system, toolchain, name, etc...)
    - Click on generate code when finished
    - CMake based project : presets ready to be used (Debug / Release) - Config will be done under the hood

2. Create empty project (optimized, no-IOC, no-HAL)
    - Configure interactively (board, name, workspace...)
    - CMake based project : presets ready to be used (Debug / Release)
    - At the end, select "Open in this window", CMake config is done under the hood when project is loaded

3. Import example
    - Located in STM32Cube/Repository ordered by board
    - `Example` : sample code to show case the HAL
    - `Example_LL` : sample code to show case "Low-Layer" (more performant HAL)
    - NB: Example apps are dropped into a folder which is named as the board name, can't be changed, so select a root project directory to place the sample in to.
    - At the end, select "Open in this window", CMake config is done under the hood when project is loaded.

### Building

- CMake style either on CLI or with VScode extension

### Flashing / Debugging

- All is bundled in the Run & Debug VSCode feature, just select:
    - ST-Link GDB server to use the in-place st-link interface
    - JLINK GDB server in case of J-Link probe (good to have the RTT logging feature)
- To be able to use semi-hosting for printf, use OpenOCD GDB Server on top of STLINK HW, see this [Semi hosting Example](https://github.com/GuiQuad06/stm32-semi-hosting)

### Test with CTest / Unity

The testing scope we want to showcase here is the unit testm so at the function level. The interest is to compile a Unity-based test app besides the STM drivers as interface library. CTest allows a seamless integration of the dedicated tests (emulated or on-target) inside the CMakelists.txt.

Example here of the [Basic Shell example](https://github.com/GuiQuad06/stm32-generic-shell).

### Interesting SW package
- FreeRTOS
- FATFS
- USB Device

### Some perso projects
- [Raw Baremetal project](https://github.com/GuiQuad06/train-barrier)
- [Fruit piano](https://github.com/GuiQuad06/fruit_piano)