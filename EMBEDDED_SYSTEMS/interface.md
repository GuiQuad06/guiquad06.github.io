---
layout: default
title: Interfaces (UART / I2C / SPI / RS-232 / RS-422 / RS-485)
---

# Serial Interfaces — Engineer's Overview

Quick reference for the three on-board serial buses (UART, I2C, SPI) and the
line drivers used to take UART off the board (RS-232, RS-422, RS-485).

> Terminology: recent specs replace *master/slave* with **controller/target** (I2C, NXP UM10204 rev 7)
> and **controller/peripheral** (SPI: MOSI/MISO → **PICO/POCI** or **SDO/SDI**). Both are used below.

---

## 1. At a glance

| | **UART** | **I2C** | **SPI** |
|---|---|---|---|
| Clocking | **Asynchronous**: no clock line, both sides agree on a baud rate | **Synchronous**: SCL driven by the controller | **Synchronous**: SCLK driven by the controller |
| Topology | **Point-to-point** (1 ↔ 1) | **Multi-controller, multi-target** shared bus | **Single controller, multiple peripherals** (star with one CS each, or daisy-chain) |
| Wires (signals) | 2: TX, RX (+ optional RTS/CTS) | 2: SDA, SCL | 3 + N: SCLK, MOSI, MISO + 1 CS per peripheral |
| + GND | Yes | Yes | Yes |
| Duplex | **Full-duplex** | **Half-duplex** (SDA bidirectional) | **Full-duplex** (Dual/Quad SPI → half-duplex) |
| Addressing | None (point-to-point) | **7-bit** (112 usable) or 10-bit address in the frame | None. Hardware **chip select** line |
| Acknowledge | No (parity bit optional) | **ACK/NACK** after each byte | No |
| Output stage | Push-pull | **Open-drain + pull-ups** (see [GPIO States](gpio_states.html)) | Push-pull (MISO tri-stated when CS is inactive) |
| Typical speed | 9600 bd – 115.2 kbd, up to a few Mbd | 100 k / 400 k / 1 M / 3.4 Mbit/s | 1 – 50 MHz typ., 100+ MHz for (Q)SPI flash |
| Useful throughput | 80 % of baud (8N1) | ~8/9 of SCL minus address/start/stop overhead | ≈ 100 % of SCLK per direction |
| Distance (logic levels) | < ~1 m (board / cable inside a box) | < ~1 m (bus capacitance ≤ 400 pF) | < ~30 cm (board level) |
| Flow control | Optional RTS/CTS or XON/XOFF | **Clock stretching** by the target | None (controller decides everything) |
| Standardized? | De facto (UART frame), electrical standards via RS-232/422/485 | Yes (NXP UM10204), derivatives SMBus, PMBus, I3C | **No** formal standard (Motorola de facto) |
| Typical use | Debug console, GNSS, BLE/Wi-Fi/cellular modules, inter-MCU | Sensors, EEPROM, PMIC, RTC, I/O expanders, config of PHYs/cameras | Flash, SD card, displays, ADC/DAC, radio transceivers, high-rate sensors |

---

## 2. UART — Universal Asynchronous Receiver/Transmitter

### 2.1 Wiring

```
   Device A              Device B
     TX  ───────────────►  RX
     RX  ◄───────────────  TX
    RTS  ───────────────►  CTS     (optional hardware flow control)
    CTS  ◄───────────────  RTS
    GND  ─────────────────  GND    (mandatory common reference)
```

- **TX and RX cross over.** This is the #1 bring-up mistake.
- Logic levels: CMOS/TTL (3.3 V or 5 V), **idle = high (mark)**.
- Variants on MCUs: **USART** (adds a clock pin for synchronous mode), single-wire half-duplex, LIN, IrDA, smartcard (ISO 7816).

### 2.2 Frame

Example: `'A'` = `0x41` = `0b0100_0001`, format **8N1**. Data is sent **LSB first**.

```
 bit     : idle| S |D0 |D1 |D2 |D3 |D4 |D5 |D6 |D7 |Sp | idle
 level 1 : ‾‾‾‾|   |‾‾‾|   |   |   |   |   |‾‾‾|   |‾‾‾| ‾‾‾‾
 level 0 :     |___|   |___|___|___|___|___|   |___|   |
```

| Field | Size | Role |
|---|---|---|
| Start bit | 1 bit, always 0 | Falling edge resynchronizes the receiver on every character |
| Data | 5–9 bits (8 typical) | LSB first |
| Parity | 0 or 1 bit (None / Even / Odd) | Detects single-bit errors |
| Stop bit(s) | 1, 1.5 or 2 bits, always 1 | Guarantees a high level before the next start bit |

Notation: `115200 8N1` = 115200 baud, 8 data bits, no parity, 1 stop bit.

### 2.3 Timing and throughput

- **Baud rate** = symbols per second. For a binary UART, baud = bit/s.
- **Efficiency**: 8N1 = 8 useful bits / 10 transmitted bits = **80 %**.
  - 115200 bd → 11 520 bytes/s.
  - 1 Mbd → 100 kB/s.
- Receivers usually **oversample ×16 (or ×8)** and sample the middle of each bit.
- **Clock tolerance**: the TX + RX baud mismatch must stay below ~±3–4 % in total. Theoretical limit is ~5 % for 8N1. Good practice is **≤ ±1.5–2 % per side**. RC oscillators (HSI) can drift beyond that over temperature, so use a crystal or calibration for high baud rates.
- Errors reported by the hardware:
  - **framing** (stop bit = 0)
  - **parity**
  - **overrun** (byte not read in time)
  - **noise** (inconsistent samples)
  - **break** (line held low for longer than a frame)
- Embedded pattern: **DMA + IDLE-line interrupt** to receive variable-length frames without a per-byte IRQ.

### 2.4 Leaving the board: UART line drivers

A UART only defines the **frame**. To go over a cable, a **transceiver** converts the logic-level TX/RX into a robust **electrical standard**. The UART peripheral and its software stay the same, so the transceiver choice is purely a **physical-layer** decision.

```
  MCU UART (3.3 V CMOS) ──► Transceiver ──► Cable (RS-232 / RS-422 / RS-485) ──► Transceiver ──► UART
```

#### RS-232 (TIA-232-F)

- **Single-ended**, referenced to GND. **Inverted, bipolar** levels:
  - logic 1 (mark) = **−3 … −15 V**
  - logic 0 (space) = **+3 … +15 V**
  - ±3 V is the undefined zone. Drivers typically output ±5 … ±12 V.
- **Point-to-point**: 1 driver, 1 receiver per line. **Full-duplex** (TX, RX, GND = minimum 3 wires).
- Standard spec: **≤ 20 kbit/s**, **~15 m** (limited by 2500 pF load). In practice, modern transceivers go up to 250 kbit/s – 1 Mbit/s over short cables.
- Transceivers: **MAX232** (5 V), **MAX3232** (3–5.5 V). An internal charge pump generates the ± voltages.
- Connector: **DE-9** (DB9). DTE pinout:
  - 2 = RXD, 3 = TXD, 5 = GND
  - 7 = RTS, 8 = CTS
  - 4 = DTR, 6 = DSR, 1 = DCD, 9 = RI
- DTE ↔ DCE uses a straight cable. DTE ↔ DTE uses a **null-modem** (crossed) cable.
- Weak points: ground-referenced, so sensitive to **ground shifts and EMI**. Short reach, only 2 nodes.

#### RS-422 (TIA-422-B)

- **Differential, balanced** pair per direction (A/B). The receiver only looks at **V(A) − V(B)**: threshold **±200 mV**, common-mode range −7 … +7 V.
- **1 driver → up to 10 receivers** per pair (multi-drop, one talker).
- **Full-duplex** with 2 twisted pairs (TX pair + RX pair, + GND reference = 5 wires).
- Up to **10 Mbit/s at ~12 m**, or **~100 kbit/s at 1200 m**.
- **Termination**: 100–120 Ω across the pair at the **receiver end** (far end of the line).
- Typical uses: long point-to-point links in noisy environments: defense/avionics sensors, NMEA 0183, PTZ cameras, industrial encoders.

#### RS-485 (TIA-485-A)

- **Differential** like RS-422, with a wider common-mode range (**−7 … +12 V**) and **tri-state drivers** (DE / /RE pins), so **several drivers can share the bus**. This makes it a true **multi-point** bus.
- Up to **32 unit loads**. 128/256 nodes are possible with 1/4- or 1/8-UL transceivers.
- **2-wire half-duplex** (most common: one pair, everyone talks in turn) or 4-wire full-duplex.
- Same speed/distance envelope as RS-422. Rule of thumb: **bit rate (bit/s) × length (m) ≤ 10⁸**.

```
  [RT]════╦═════════════╦═════════════╦══════════[RT]
          ║             ║             ║
       ┌──╨───┐      ┌──╨───┐      ┌──╨───┐
       │Node 1│      │Node 2│      │Node N│
       └──────┘      └──────┘      └──────┘
  Twisted pair A/B (+ GND reference). Daisy-chain with short stubs.
  RT = 120 Ω termination at BOTH ends of the bus only.
```

- Design checklist:
  - **Fail-safe biasing**: when no driver is active, the differential voltage is ~0 V and receivers may output garbage. Add bias resistors (pull-up on the non-inverting line, pull-down on the inverting line, at one point of the bus) or use transceivers with built-in true fail-safe.
  - **A/B naming is inconsistent** between the TIA standard and some vendors. If nothing works, swap A and B.
  - **Direction control** (half-duplex): enable DE before transmitting, and release it only after the **last stop bit has left the shift register**. Use the UART *Transmission Complete* flag, not *TX register empty*. Hardware DE support (STM32 USART DE pin) or Linux `TIOCSRS485` (`SER_RS485_ENABLED`) handles this automatically.
  - **No built-in arbitration**: the protocol must avoid collisions, usually with **master/slave polling**.
  - Long lines between buildings or vehicles: use **galvanic isolation** (isolated transceivers, e.g., ADM2587E) and TVS/surge protection.
- Protocols on top: **Modbus RTU**, Profibus DP, DMX512, BACnet MS/TP, many proprietary sensor buses.

> **Multi-protocol transceivers** (e.g., MAX3160E, SP339) can be switched between RS-232 / RS-422 / RS-485 by software or a strap pin. This is how rugged computers offer "RS-232/422/485 configurable" serial ports.

#### Comparison

| | **UART (TTL/CMOS)** | **RS-232** | **RS-422** | **RS-485** |
|---|---|---|---|---|
| Signaling | Single-ended, 0 / 3.3–5 V | Single-ended, ±3…15 V (inverted) | Differential | Differential |
| Topology | Point-to-point | Point-to-point | 1 driver → 10 receivers | Multi-point bus, 32 UL (up to 256) |
| Duplex | Full | Full | Full (2 pairs) | Half (2-wire) or Full (4-wire) |
| Min. wires | TX, RX, GND | TX, RX, GND | 2 pairs + GND | 1 pair + GND (2-wire) |
| Max distance | < ~1 m | ~15 m | 1200 m | 1200 m |
| Max speed | Few Mbd (board) | 20 kbit/s (spec), ~1 Mbit/s (practice) | 10 Mbit/s (short) | 10 Mbit/s (short), 50+ Mbit/s parts exist |
| Noise immunity | Poor | Low (ground-referenced) | High (differential) | High (differential) |
| Termination | No | No | 100–120 Ω at receiver | 120 Ω at both bus ends + fail-safe bias |
| Software impact | — | None (transparent) | None (transparent) | **Direction control + collision-free protocol** |

#### Motivation: which one to pick?

| Choose… | When… | Watch out for |
|---|---|---|
| **TTL UART** (no transceiver) | Both chips are on the same board or in the same box, short cable, common ground. Debug console, MCU ↔ GNSS/BLE/modem module | 3.3 V vs 5 V levels, no protection, very sensitive to noise on cables |
| **RS-232** | Connecting to a **PC, legacy equipment, lab instruments or network-gear console ports**. Point-to-point, short cable (< 15 m), low speed. DB9 is everywhere and USB-RS232 adapters are cheap | Ground loops, EMI, low speed, only 2 nodes, needs a ±V charge pump |
| **RS-422** | **Long or noisy** point-to-point link, **full-duplex**, or **one talker / several listeners** (e.g., one GNSS broadcasting to several computers). Higher speed than RS-232 | 4 wires + GND, termination, no multi-talker capability |
| **RS-485** | **Several nodes on one cable** (sensors, actuators, drives), any node can transmit, long distance, cheapest wiring (one twisted pair). Industrial fieldbus (Modbus RTU, Profibus, DMX) | Half-duplex turnaround timing, termination at both ends, fail-safe bias, the protocol must handle bus access |

---

## 3. I2C — Inter-Integrated Circuit

### 3.1 Wiring

```
        VDD     VDD
         │       │
        [Rp]    [Rp]
         │       │
SDA ─────┴───┬───┼────────┬────────────┬─────────
SCL ─────────┼───┴──┬─────┼─────┬──────┼─────┬───
             │      │     │     │      │     │
           ┌─┴──────┴─┐ ┌─┴─────┴─┐  ┌─┴─────┴─┐
           │Controller│ │ Target  │  │ Target  │
           │  (MCU)   │ │  0x48   │  │  0x50   │
           └──────────┘ └─────────┘  └─────────┘
```

- **2 wires**: SDA (data, bidirectional) and SCL (clock), plus GND.
- **Open-drain**: devices only pull lines **low**. External **pull-up resistors** bring them high. This makes **wired-AND** possible, which enables arbitration and clock stretching (see [GPIO States](gpio_states.html)).
- **Multi-target**: each target has an address. **Multi-controller** is possible thanks to **arbitration**: a controller that sends a 1 but reads back a 0 loses and backs off.
- **Half-duplex**: one data line, the direction changes within the transaction.

### 3.2 Transaction

```
 Write register:  S | ADDR(7) W | A | REG | A | DATA | A | ... | P
 Read register :  S | ADDR(7) W | A | REG | A | Sr | ADDR(7) R | A | DATA | A | ... | DATA | N | P

 S = START (SDA ↓ while SCL high)    P = STOP (SDA ↑ while SCL high)    Sr = repeated START
 A = ACK (receiver pulls SDA low)    N = NACK (controller ends the read)
```

- Data is sent **MSB first**. SDA may only change while **SCL is low**. START and STOP are the only exceptions.
- **9 clocks per byte** (8 data + ACK).
- **Addressing**: 7-bit gives 128 addresses, of which 16 are reserved (`0x00–0x07`, `0x78–0x7F`), leaving **112 usable**. 10-bit addressing uses the `11110xx` prefix.
- **Clock stretching**: a slow target holds SCL low to make the controller wait.
- **Bus stuck** (SDA held low after a reset mid-transfer): the controller toggles SCL up to **9 times**, then sends a STOP.

### 3.3 Speed modes and throughput

| Mode | SCL max | Pull-up sink current (IOL) | Bus capacitance max | Raw max payload* |
|---|---|---|---|---|
| Standard-mode (Sm) | 100 kHz | 3 mA | 400 pF | ~11 kB/s |
| Fast-mode (Fm) | 400 kHz | 3 mA | 400 pF | ~44 kB/s |
| Fast-mode Plus (Fm+) | 1 MHz | 20 mA | 550 pF | ~110 kB/s |
| High-speed (Hs) | 3.4 MHz | (current-source pull-up) | 100–400 pF | ~380 kB/s |
| Ultra-fast (UFm) | 5 MHz | push-pull, **write-only, no ACK** | — | — |

\* SCL / 9, before address, START/STOP and register-pointer overhead. A typical "read 2 bytes from a register" costs ~5 bytes on the wire.

### 3.4 Pull-up sizing

- **Minimum** (sink-current limit): `Rp_min = (VDD − VOL_max) / IOL` → 3.3 V, 0.4 V, 3 mA ≈ **970 Ω**.
- **Maximum** (rise-time limit): `Rp_max = tr / (0.8473 × Cb)`, where tr is 1000 ns (Sm), 300 ns (Fm) or 120 ns (Fm+).
  - Example: Fm, Cb = 100 pF → ≈ **3.5 kΩ**.
- Typical values: **4.7 kΩ @ 100 kHz**, **2.2 kΩ @ 400 kHz**. Use **one pair of pull-ups per bus segment**, not one per device.
- Mixed voltages: use a level shifter (MOSFET-based such as BSS138, or PCA9306). Longer distances: use bus buffers / differential extenders (P82B715, PCA9615).

### 3.5 Family

- **SMBus / PMBus**: I2C subset for power management. 10–100 kHz, 35 ms timeout, optional PEC (CRC-8).
- **I3C** (MIPI): successor. Push-pull, 12.5 MHz SDR, in-band interrupts, dynamic addressing, backward compatible with most I2C targets.

---

## 4. SPI — Serial Peripheral Interface

### 4.1 Wiring

```
                SCLK ───┬────────────┬───────
                MOSI ───┼──┬─────────┼──┬────
  Controller    MISO ───┼──┼──┬──────┼──┼──┬─
                        │  │  │      │  │  │
                     ┌──┴──┴──┴─┐ ┌──┴──┴──┴─┐
                CS1 ─┤ Periph 1 │ │ Periph 2 ├─ CS2
                     └──────────┘ └──────────┘
          (CS1 / CS2 are both driven by the controller, active low)
```

| Signal | Also called | Direction |
|---|---|---|
| SCLK | SCK, CLK | Controller → peripheral |
| MOSI | **PICO**, SDO (controller side), SDI (peripheral side), DIN | Controller → peripheral |
| MISO | **POCI**, SDI (controller side), SDO (peripheral side), DOUT | Peripheral → controller |
| CS | SS, /CS, nCS, CE (active low) | Controller → one peripheral |

- **One controller**, N peripherals. **Wires = 3 + N** (one CS per peripheral). Alternatively, a **daisy-chain** where MISO of one feeds MOSI of the next and all share one CS (shift registers, LED drivers).
- **Full-duplex**: each clock shifts one bit out on MOSI **and** one bit in on MISO. The two shift registers form a ring. To read, the controller must clock out dummy bytes.
- No addressing, no ACK, no flow control: **the CS line selects the device and frames the transaction**. Many devices need CS to toggle between commands.
- A peripheral must **tri-state MISO** when its CS is high. Otherwise, add a buffer.

### 4.2 Clock modes (CPOL / CPHA)

| Mode | CPOL (idle clock) | CPHA | Data sampled on | Data shifted on |
|---|---|---|---|---|
| 0 | 0 (low) | 0 | Rising edge | Falling edge |
| 1 | 0 (low) | 1 | Falling edge | Rising edge |
| 2 | 1 (high) | 0 | Falling edge | Rising edge |
| 3 | 1 (high) | 1 | Rising edge | Falling edge |

Mode 0 and mode 3 are the most common. **Check the peripheral datasheet.** A wrong mode gives data shifted by one bit, which is a classic bug. Word size is usually 8 or 16 bits, **MSB first**.

### 4.3 Throughput and variants

- **1 bit per clock per direction**, with almost no protocol overhead.
  - 20 MHz → 2.5 MB/s each way.
- Limits: trace length and **round-trip delay on MISO** (SCLK out → peripheral clock-to-out → MISO back must fit within half a period), capacitance, isolators. Series termination resistors (22–33 Ω) help on fast edges.
- **3-wire SPI**: single bidirectional SDIO line, so half-duplex.
- **Dual / Quad / Octal SPI (QSPI/OSPI)**: 2 / 4 / 8 bidirectional data lines, half-duplex, used for **NOR flash** and memory-mapped XIP.
  - Quad @ 100 MHz → 50 MB/s.

---

## 5. Motivation: UART vs I2C vs SPI (vs RS-xxx)

| Choose… | When… | Avoid when… |
|---|---|---|
| **UART** | Point-to-point link between two **independent processors/modules** (GNSS, BLE/Wi-Fi/cellular modem, other MCU). Human-readable **debug console**. No shared clock available. Easy to extend over cable with RS-232/422/485 | You need many devices on the same lines, or high throughput with exact timing |
| **I2C** | **Many low-speed devices** (sensors, EEPROM, RTC, PMIC, I/O expander, camera/PHY configuration) with only **2 pins**. Need for an ACK / device presence detection. Multi-controller | High data rates (> ~100 kB/s), long cables, noisy environments, many devices with the same fixed address (needs a mux like TCA9548A) |
| **SPI** | **Throughput and simplicity**: flash, SD card, displays, fast ADC/DAC, radio transceivers, IMU at high ODR. Full-duplex streaming, deterministic timing, easy DMA | Pin count matters (1 CS per device), long distance, need for addressing/ACK, hot-plug |
| **RS-232** | Interop with **PC / legacy / instruments**, short point-to-point cable | Distance > 15 m, noisy environments, > 2 nodes |
| **RS-422** | **Long-distance, noisy, full-duplex** point-to-point, or one talker to several listeners | Several devices must transmit |
| **RS-485** | **Multi-node field bus** over one twisted pair, long distance, industrial protocols (Modbus RTU…) | You can't manage direction control / bus access in software |

Rule of thumb:
- **inside the chip neighbourhood** → SPI (speed) or I2C (pin count)
- **between boards / modules** → UART
- **between boxes** → UART + RS-232/422/485 (or CAN / Ethernet when the protocol must handle arbitration or higher bandwidth)

[Back to Embedded Systems](./)
