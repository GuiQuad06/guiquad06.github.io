---
layout: default
title: Memory Footprint
---

# Firmware Memory Sections

Reference notes on the standard binary sections found in embedded firmware images.

## 1. `.text` Section

**Purpose**

- Contains the **executable instructions** (machine code) of the program.
- This is where the actual logic of your firmware resides.

**Characteristics**

- **Read-only** during execution (to prevent accidental or malicious code modification).
- **Executable** (marked as such in the section flags).
- Typically placed in **flash memory** in embedded systems.
- May be subdivided into functions or code blocks by the linker.

**Example Contents**

- Main program logic
- Function implementations
- Interrupt service routines (ISRs)

## 2. `.data` Section

**Purpose**

- Stores **initialized global and static variables**.
- These variables have a predefined value at compile time.

**Characteristics**

- **Read-write** (can be modified during runtime).
- **Occupies space in the binary file** (values are stored in the firmware image).
- Typically placed in **RAM** at runtime, but initialized from flash at startup.

**Example Contents**

- Global variables with initial values (e.g., `int counter = 10;`)
- Static variables with initial values
- Lookup tables or constants that need to be in RAM

## 3. `.bss` Section

**Purpose**

- Stores **uninitialized global and static variables**.
- These variables are guaranteed to start with a value of zero.

**Characteristics**

- **Read-write** (can be modified during runtime).
- **Does not occupy space in the binary file** (only a size is recorded; the section is
  zero-initialized at runtime).
- Typically placed in **RAM** at runtime.

**Example Contents**

- Global variables without initial values (e.g., `int buffer[100];`)
- Static variables without initial values
- Large buffers or arrays that don't need initial values

## 4. `.rodata` Section

**Purpose**

- Contains **read-only data**, such as constants and string literals.
- This data cannot be modified during runtime.

**Characteristics**

- **Read-only** (to prevent accidental or malicious modification).
- **Occupies space in the binary file** (values are stored in the firmware image).
- Typically placed in **flash memory** in embedded systems.

**Example Contents**

- String literals (e.g., `"Hello, World!"`)
- Constants defined with `const` (e.g., `const float PI = 3.14159;`)
- Lookup tables or other read-only data structures

## Mapping Between Notations

Section naming across toolchains:

| ELF/GNU   | ARM Keil      |
| --------- | ------------- |
| `.text`   | `CODE`        |
| `.rodata` | `RO / CONST`  |
| `.data`   | `RW`          |
| `.bss`    | `ZI`          |

[Back to Embedded Systems](./)
