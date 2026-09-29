---
layout: default
title: ARM Assembly
---

# ARM Assembly

Sources:
[An Overview of the ARM Assembly Language Instruction Set](https://www.youtube.com/watch?v=GBRdzaAxHB8) ·
[Assembly Language Programming with ARM – Full Tutorial for Beginners](https://www.youtube.com/watch?v=gfmRrPjnEw4)

## Overview

RISC:

- Small instruction set
- 1 clock cycle per instruction

**64-bit ARM CPU:**

- 64-bit memory addressing
- 32-bit instructions
- Multi-core support
- Little endian

## General syntax

```armasm
Opcode dest, src
```

| Instruction | Meaning |
|---|---|
| `MOV` | Move data from a register to another |
| `MVN` | Move and negate |
| `LDR` | Load data from memory into a register |
| `STR` | Store a register into a memory / peripheral address |

```armasm
LDR R1, [R0]        ; brackets dereference the value at the wanted address
LDR R2, [R0, #4]    ; offset of 4 bytes
STR R1, [R0]        ; store R1 into the address held by R0
```

`.data` declares the data section.

## Arithmetic & logic

```armasm
ADD / SUB / MUL
SUBS                ; same as SUB but also sets the CPSR flags

AND / ORR / EOR
LSL                 ; logical shift left
LSR                 ; logical shift right
ROR                 ; rotate right

MOV R1, R0, LSL #2  ; left shift R0 by 2 (x4) and move the result into R1
```

## Conditions & branches

```armasm
CMP R0, R1          ; computes R0 - R1 and sets the flags
```

| Branch | Condition |
|---|---|
| `BLT` | less than |
| `BLE` | less than or equal |
| `BEQ` | equal |
| `BNE` | not equal |
| `BGT` | greater than |
| `BAL` | always |
| `BL` | branch & link — LR is loaded so we can return to the caller context |
| `BX LR` | return to the address held by LR |

Conditional execution can also be suffixed on regular instructions:
`ADDLT` (add if less than), `MOVGE` (move if greater or equal).

### Careful: fall-through

If you branch to `greater` but the `default` label sits right after it in the
sequence, execution will fall into `default` anyway.

Rule of thumb: **first the branches, last the labels** in the code section.

```armasm
_start:
    MOV R0, #2
    MOV R1, #1

    CMP R0, R1
    BGT greater
    BAL default

default:
    MOV R2, #40

greater:
    MOV R2, #30
```

## Stack

```armasm
PUSH {R0, R1}       ; push onto the stack (POP to get the values back)
```

The stack pointer holds the address of the stack.

## Registers

- `R0`–`R12` — general purpose, 32-bit
  - `R7` — syscall number
- `SP` — stack pointer (points into RAM)
- `LR` — link register, handles the function return address
- `PC` — program counter, the instruction the CPU is on
- `CPSR` — `NZCVI` flags
  - `N` negative
  - `Z` zero
  - `C` carry
  - `V` overflow
- `SPSR` — saved program status register

## Loops

```armasm
.global _start
.equ endlist, 6

_start:
    LDR R0, =list
    LDR R3, =endlist
    LDR R1, [R0]
    ADD R2, R2, R1

loop:
    LDR R1, [R0, #4]!
    CMP R1, R3
    BEQ exit
    ADD R2, R2, R1
    BAL loop

exit:

.data
list:
    .word 1, 2, 3, 4, 5, 6
```

## Functions

```armasm
.global _start

add2:
    ADD R2, R0, R1
    BX LR

_start:
    MOV R0, #3
    MOV R1, #5
    BL add2
    MOV R3, #4
```

[Back to Languages](./)
