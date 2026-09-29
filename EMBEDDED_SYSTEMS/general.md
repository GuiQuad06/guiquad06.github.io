---
layout: default
title: Embedded Systems — General
---

# General

## Embedded HW Part

| Situation | Configuration |
|---|---|
| Fast unidirectional digital signal (SPI, clock) | Push-pull, no pull, drive strength matched to load |
| Shared bus, multiple talkers, mixed voltages (I2C, 1-Wire) | Open-drain + pull-up |
| Multiple devices signalling one interrupt line | Open-drain + pull-up (wired-AND, active low) |
| Bidirectional parallel bus | Tri-state push-pull with strict OE arbitration |
| Button to GND | Input + pull-up (internal is fine) |
| Button to VDD | Input + pull-down |
| MOSFET gate / enable that must be safe at power-up | **External** pull-down — do not rely on firmware |
| Unused pin | Output driven low, or input with a pull enabled — never floating |
| Analog pin | Analog mode, all buffers and pulls off |

- Open Drain w/ Pull-up (NMOS only)
- Inputs as floating (High-Z)
- Push-Pull for fast transactions

> See [GPIO States (HW)](gpio_states.html) for the full engineer's overview of these pad
> structures.

### Logic Analyzer

- **Saleae** — 1000$, weak reliability (avoid for CI/CD), but good SW + SDK support for
  HLA + analyzers.
- **LabJack** — 260$, more reliable, RJ45 connection.

---

## Memory Footprint (quick reference)

- Total RO (CODE + RO Data) => flash
- Total RW (RW Data + ZI Data) => RAM
- Total ROM (CODE + RO Data + RW Data) => needed ROM footprint — do not explicitly
  initialize a variable to 0 just to force it into `.bss`.

| Section | Content |
|---|---|
| CODE | `.text` — the firmware, the instructions |
| RO | `.rodata` — const & string literals |
| RW | `.data` — explicitly initialized variables |
| ZI | `.bss` — uninitialized global & static variables, not stored in the binary file |

> See [Memory Footprint](memory_footprint.html) for the detailed per-section reference.

---

## Misc

- Avoid `float` as much as possible.
- `volatile` — informs the compiler that a variable may be modified by other means
  (an interrupt or a piece of hardware); prevents unwanted optimizations on code using
  that variable.
  - Compiler optimizations may otherwise remove, for example, a conditional branch that
    seemingly has no chance of being taken — but if the variable can be updated by
    hardware, `volatile` stops the compiler from making that assumption.
- **LUT (lookup tables)** — less processing / more memory space.
  - Always add the `const` qualifier so the LUT is allocated only in ROM (not twice).
- **Macro functions** — basic stuff (`min` / `max`).
- **Inline functions**:
  - `inline` keyword — suggests the compiler inline the function.
  - `__attribute__((always_inline))` on the prototype — forces inlining (faster call,
    slightly bigger `.text`).

[Back to Embedded Systems](./)
