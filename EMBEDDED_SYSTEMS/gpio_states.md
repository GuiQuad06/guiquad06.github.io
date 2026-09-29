---
layout: default
title: GPIO States (HW)
---

# GPIO / IO States — Engineer's Overview

Reference notes on digital pad structures: push-pull (incl. tri-state), open-drain,
high-impedance, pull-up and pull-down.

---

## 1. The core building block: two transistors

Every digital output pin is built from (at most) two switches between the supply rails:

```
        VDD
         |
      [ P-MOS ]   <- high-side / pull-up device
         |
         +------- PAD (physical pin)
         |
      [ N-MOS ]   <- low-side / pull-down device
         |
        GND
```

Which device exists (or is enabled) defines the output type.

---

## 2. Push-Pull and Tri-State

### Push-pull

Both devices present, exactly one on at a time.

| Logic level | P-MOS | N-MOS | Pad is |
|---|---|---|---|
| 1 | ON | OFF | actively driven to VDD |
| 0 | OFF | ON | actively driven to GND |

- The pin *pushes* current out when high, *pulls* current in when low.
- Fast edges, no external resistor needed, roughly symmetric drive.
- **Cannot be wired to another active output.** One driving high against one driving
  low creates a low-resistance VDD→GND path: hundreds of mA, latch-up, dead silicon.
- Default choice for SPI, UART TX, clocks, LED drive, chip selects, reset outputs.

Both transistors on simultaneously is called **shoot-through**. Real drivers insert
dead-time / break-before-make logic to prevent it during transitions.

### Tri-state = push-pull + output enable

Tri-state is not a separate structure — it is a push-pull driver with an
**output-enable (OE)** control, giving three reachable states:

1. Driven high
2. Driven low
3. High-Z (released)

```
            OE
             |
   DATA --[ driver ]-- PAD
```

This is what makes **shared / bidirectional buses** possible while keeping push-pull
speed: external memory buses, FPGA data buses, backplanes. Exactly one device asserts
OE at a time; the rest float.

Critical concern: **bus turnaround**. When handing the bus from A to B, A must release
*before* B drives, or you get contention. Hardware handles this with bus-hold / keeper
cells or a guaranteed idle cycle in the protocol.

> Terminology note: an open-drain pin is also high-Z when idle, but it is **not**
> tri-state — it has no drive-high capability at all. Tri-state implies all three
> states are reachable.

---

## 3. Open Drain (Open Collector in bipolar)

Only the **N-MOS** exists or is enabled; the P-MOS is removed or permanently off.

| Logic level | Pad is |
|---|---|
| 0 | actively driven to GND |
| 1 | **released** — floating, high-Z |

The pin can only pull *down*. A logic high requires an external (or internal)
**pull-up resistor** to a rail.

```
              VCC_bus
                 |
                [ R ] pull-up
                 |
   pin A --------+-------- pin B -------- pin C
     |           |            |
   [NMOS]     receiver     [NMOS]
```

### Key properties

- **Wired-AND**: the bus is high only if *every* participant releases it. Any device
  pulling low wins — contention is structurally impossible. That is the whole point.
- **Voltage-domain translation**: the pull-up rail need not equal the chip's VDD. A
  1.8 V MCU can drive a 3.3 V open-drain bus provided the pad is tolerant to that rail.
  Common mixed-voltage trick.
- **Asymmetric edges**: falling edge is fast (active transistor); rising edge is an RC
  exponential, `τ = R × C_bus`. This directly limits bus speed.
- **Static current**: while the bus is held low, `I = V_CC / R` flows continuously.
  Power vs. speed trade-off when choosing R.

### Typical uses

I²C (SDA/SCL), SMBus, 1-Wire, shared interrupt lines (several sensor `nIRQ` outputs
tied together), reset lines, fault/alarm signals.

### Sizing the pull-up

- Too large → slow rise time, may violate the bus spec.
- Too small → the low-side transistor cannot sink enough current to reach
  `V_OL,max` (I²C: 3 mA sink, `V_OL <= 0.4 V`).

```
R_min = (V_CC - 0.4 V) / 3 mA
R_max = t_r / (0.8473 * C_bus)
```

Practically: 4.7 kΩ at 100 kHz, 2.2 kΩ or 1 kΩ at 400 kHz / 1 MHz.

---

## 4. High Impedance (High-Z)

Not an output mode — a *state*. Both transistors off, the pad is connected to
essentially nothing. Impedance in the MΩ–GΩ range, leakage typically < 1 µA.

Electrically the pin is **floating**: its voltage is undefined and set by whatever else
is on the net (another driver, a resistor, leakage, ambient capacitive coupling).

Where High-Z appears:

- Any pin configured as an **input**.
- An open-drain output driving a logic 1.
- A push-pull output with **OE** deasserted (tri-state).
- After reset, before firmware configures the GPIO — most MCUs boot with pins as
  high-Z inputs.

### The floating-input hazard

A CMOS input left floating sits near the switching threshold. Both transistors of the
receiving inverter conduct partially → **crowbar current**, elevated consumption, and
the input oscillates from noise pickup, generating spurious interrupts.

Classic low-power bug: the product draws 300 µA instead of 2 µA because three unused
pins are floating.

**Rule: never leave a CMOS input floating.** Configure unused pins as outputs driven
low, or as inputs with an internal pull enabled.

---

## 5. Pull-Up

A resistor from the pin to VDD — external discrete, or a weak internal device inside
the pad. Defines a **default logic 1** when nothing actively drives the net.

- **External**: typically 1 k–10 kΩ. Precise, known value, works even when the chip is
  unpowered or in reset.
- **Internal**: usually 20–100 kΩ and poorly specified (e.g. 20 k min / 50 k typ / 100 k
  max). Fine for buttons and idle levels; **not** adequate for I²C, too weak for noisy
  or long traces.

Uses: button inputs (button shorts to GND), open-drain bus termination, holding a
reset/enable asserted, boot-strap pins.

## 6. Pull-Down

Same idea, resistor to GND, defines a **default logic 0**.

Uses: enable pins that must default to disabled, boot-strap/config straps, discharging
a MOSFET gate so a load never turns on inadvertently.

### Caveats for both

- Internal pulls are typically **disabled during and shortly after reset**. Anything
  that must be at a known level at power-up (MOSFET gate, reset line, boot-mode strap)
  needs an **external** resistor — firmware runs too late.
- Internal pulls behave differently in sleep / deep-sleep; many MCUs need explicit
  retention configuration to keep them active.
- Never fit a pull-up and pull-down of similar value on the same net unless you want a
  divider — you'll sit at mid-rail.
- A pull on a push-pull-driven pin just wastes current; pulls are only meaningful on
  inputs, open-drain outputs, or tri-stated nets.

---

## 7. Putting the chain together

A typical MCU pad cell contains all of it:

```
                      VDD
                       |
                   [ PU res ]---o  (pull-up enable bit)
                       |
   OUT_DATA --+--[ PMOS ]
              |        |
   OE --------+        +-----o<PAD>o-----> ESD diodes, VDD/GND clamps
              |        |
              +--[ NMOS ]                 |
                       |                  +--> [ Schmitt input buffer ] --> IN_DATA
                   [ PD res ]---o             (input enable bit)
                       |
                      GND
```

Configuration registers you will meet, whatever the vendor calls them:

| Register concept | Controls |
|---|---|
| MODE / DIR | input vs. output vs. alternate function vs. analog |
| OTYPE | push-pull vs. open-drain |
| PUPD | pull-up / pull-down / none |
| OSPEED / DRIVE | slew rate and drive current (2 / 4 / 8 / 20 mA) |
| Input enable | disconnects the input buffer — essential for analog pins and sleep leakage |

**Analog mode** disables *both* the output drivers and the digital input buffer.
Required for ADC/DAC pins and often the lowest-leakage state on the part.

---

## 8. Decision guide

| Situation | Configuration |
|---|---|
| Fast unidirectional digital signal (SPI, clock) | Push-pull, no pull, drive strength matched to load |
| Shared bus, multiple talkers, mixed voltages | Open-drain + pull-up |
| Multiple devices signalling one interrupt line | Open-drain + pull-up (wired-AND, active low) |
| Bidirectional parallel bus | Tri-state push-pull with strict OE arbitration |
| Button to GND | Input + pull-up (internal is fine) |
| Button to VDD | Input + pull-down |
| MOSFET gate / enable that must be safe at power-up | **External** pull-down — do not rely on firmware |
| Unused pin | Output driven low, or input with a pull enabled — never floating |
| Analog pin | Analog mode, all buffers and pulls off |

---

## 9. Practical gotchas

1. **Open-drain rise time** dominates I²C timing failures. If the bus is marginal,
   scope SDA and look at the RC curve before blaming firmware.
2. **Reset state ≠ configured state.** Between power-on and `GPIO_Init()` pins are
   inputs. Anything that must be safe in that window needs external hardware.
3. **Drive strength affects EMI.** Faster edges = more HF content = more radiated
   emissions and ground bounce. Use the lowest setting that meets timing.
4. **5 V tolerance is not universal.** A pin with an ESD clamp diode to VDD conducts
   above VDD + 0.7 V. Open-drain to a higher rail only works on explicitly tolerant
   pads.
5. **Sleep-mode leakage** is usually floating inputs, enabled pull-ups fighting an
   external driver, or input buffers left on for analog pins.
6. **Simultaneous switching noise**: many push-pull outputs toggling together cause
   supply/ground bounce. Stagger them or reduce drive strength.
7. **Bus turnaround** on tri-state buses: release before drive, always.

[Back to Embedded Systems](./)
