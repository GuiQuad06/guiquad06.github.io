---
layout: default
title: CMake
---

# CMake

> See the tutorial in the CMake repo!

## Skeleton (state of the art, `INTERFACE` target)

```cmake
cmake_minimum_required(VERSION 3.25)
project(toto C ASM)

# Reset build config flags
set(CMAKE_C_FLAGS_DEBUG "")
set(CMAKE_C_FLAGS_RELEASE "")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "")
set(CMAKE_C_FLAGS_MINSIZEREL "")

# Common interface library
add_library(common_configuration INTERFACE)

target_compile_definitions(common_configuration INTERFACE
  $<$<CONFIG:Debug>:DEBUG>
  $<UPPER_CASE:${BOARD}>
  EZAIRO8300
  EZAIRO8300_CM3
  GIT_COMMIT_ID="${GIT_COMMIT_ID}"
  MAJOR=${MAJOR}
  MINOR=${MINOR}
  REV=${REV}
  SK5_CID=101
)

target_compile_options(common_configuration INTERFACE
  $<$<CONFIG:Release>:-O2>
  $<$<CONFIG:RelDeb>:-O2>
  $<$<CONFIG:RelDeb>:-ggdb3>
  $<$<CONFIG:Debug>:-O1>
  $<$<CONFIG:Debug>:-ggdb3>
  -Wall
  -fdata-sections
  -fdiagnostics-color=always
  -ffunction-sections
  -fmessage-length=0
  -fsigned-char
  -std=gnu11
)

target_include_directories(common_configuration INTERFACE
  ${EZAIRO8300_CALIB_DIR}/include
  ${EZAIRO8300_SDK_DIR}/include/cm3
  ${EZAIRO8300_SDK_DIR}/include/cm3/cmsis_drivers
  ${EZAIRO8300_SDK_DIR}/include/shared
  ${EZAIRO8300_SDK_DIR}/source/shared/calibratelib/include
  ${EZAIRO8300_SDK_DIR}/source/shared/nvmlib/include
  ${NEXTGEN_SRC_DIR}/BSP/inc/${BOARD}
  ${NEXTGEN_SRC_DIR}/cfx_cm3_communication/inc
  ${RTOS_DIR}/include
  ${RTOS_DIR}/portable/GCC/ARM_CM3
  ../common/include
  include
)

target_link_options(common_configuration INTERFACE
  -Tsections.ld
  -Wl,--gc-sections
  -Wl,-Map=${CM3_APP}.map
  -nostartfiles
)

target_link_directories(common_configuration INTERFACE
  .
)

set(MAIN_APP_SRC
  ${EZAIRO8300_SDK_DIR}/source/Cortex-M3/cmsis/source/GCC/startup_sk5.S
  ${EZAIRO8300_SDK_DIR}/source/Cortex-M3/cmsis/source/sbrk.c
  ${EZAIRO8300_SDK_DIR}/source/Cortex-M3/cmsis/source/system_sk5.c
  ../common/src/shared_memory.c
  src/irq_handlers.c
  src/main.c
)

### Specific image ###################################
add_executable(specific_image ${MAIN_APP_SRC})

target_link_libraries(specific_image PRIVATE
  common_configuration
)

target_compile_definitions(specific_image PRIVATE
  EZ_FW_TYPE=0x0000
)

set_target_properties(specific_image PROPERTIES
  LINK_DEPENDS ${LINKER_SCRIPT_PATH}   # allows re-linking if the linker script is modified
  OUTPUT_NAME ${CM3_APP}
  SUFFIX ${SUFFIX}
)

add_custom_command(TARGET specific_image POST_BUILD
  COMMAND ${ARM_NONE_EABI_DIR}/arm-none-eabi-objdump -dSs ${CM3_APP}${SUFFIX} > ${CM3_APP}.dis
)
#######################################################
```

[Back to Build Flow](./)
