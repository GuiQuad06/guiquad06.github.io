---
layout: default
title: C Language
---

# C Language

- [File skeleton](#file-skeleton)
- [Standard headers cheatsheet](#standard-headers-cheatsheet)
- [Strings](#strings)
- [Structures: padding, packing, bit fields](#structures-padding-packing-bit-fields)
- [Function pointers](#function-pointers)
- [External linkage](#external-linkage)
- [Embedded guidelines](#embedded-guidelines)
- [Driver / HAL / CMSIS](#driver--hal--cmsis)
- [printf in an embedded system](#printf-in-an-embedded-system)
- [`static inline` benchmark](#static-inline-benchmark)

---

## File skeleton

**Header:**

```c
/* Includes ------------------------------------------------------------------*/
/* Exported constants --------------------------------------------------------*/

/* Exported macros -----------------------------------------------------------*/

/* Exported types ------------------------------------------------------------*/

/* Exported variables --------------------------------------------------------*/

/* Exported functions prototypes ---------------------------------------------*/
```

**Source:**

```c
/* Includes ------------------------------------------------------------------*/

/* Private types -------------------------------------------------------------*/

/* Private constants ---------------------------------------------------------*/

/* Private macros ------------------------------------------------------------*/

/* Private variables ---------------------------------------------------------*/

/* Private function prototypes -----------------------------------------------*/

/* Exported variables definition ---------------------------------------------*/

/* Function definition -------------------------------------------------------*/

/*************************/
/*   PUBLIC FUNCTIONS    */
/*************************/

/*************************/
/*   PRIVATE FUNCTIONS   */
/*************************/
```

---

## Standard headers cheatsheet

| Header | Content |
|---|---|
| `<assert.h>` | `assert()` |
| `<ctype.h>` | `isalnum`, `isalpha`, `islower`, `isupper`, `isspace`, `isdigit` |
| `<math.h>` | maths |
| `<stdbool.h>` | `bool`, `true`, `false` |
| `<stdio.h>` | `printf`, `scanf`, `fprintf`, `sprintf`, `puts`, `gets`, `fopen`, `fread`, `fwrite`, `getchar`, `putchar` |
| `<string.h>` | `memcpy`, `memset`, `strlen`, `strstr`, `strcmp`, `strncmp`, `strchr`, `strnchr`, `strcspn` — **banned:** `strcat`, `strcpy`, `strtok` |
| `<stdlib.h>` | `EXIT_SUCCESS`, `EXIT_FAILURE`, `atof`, `atoi`, `atol`, `atoll`, `malloc`, `calloc`, `free`, `getenv`, `system`, `abs`, `bsearch`, `rand`, `strtol` |

---

## Strings

### Find (count) a character

```c
int cpt = 0;
char *ptr = string;

while ((ptr = strchr(ptr, ch)))
{
    ptr++;
    cpt++;
}
return cpt;
```

### Remove a character

```c
char *ptr = string;

while ((ptr = strchr(ptr, ch)))
{
    strcpy(ptr, ptr + 1);
}
```

### `strtol` (string to long)

```c
long value = strtol(str, &end_ptr, 10);
assert(str != end_ptr);
str = end_ptr;
```

Combine `strtol` with `strtok` to avoid keeping a character index, hence an extra loop.

### Parsing a `"%d%s%d"` string into a structure

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

typedef struct {
    int   studentNumber;
    char *studentName;
    int   grade;
} STUDENT;

int main(void)
{
    STUDENT *student = malloc(sizeof(STUDENT));
    student->studentName = malloc(201);  // 200 characters + '\0'
    char *eptr;

    char t1[] = "185213,Example Name,88";
    char *token = strtok(t1, ",");              // Read studentNumber
    student->studentNumber = strtol(token, &eptr, 10);
    token = strtok(NULL, ",");                  // Next token
    strcpy(student->studentName, token);
    token = strtok(NULL, ",");                  // Next token
    student->grade = strtol(token, &eptr, 10);

    printf("%d\n", student->studentNumber);
    printf("%s\n", student->studentName);
    printf("%d\n", student->grade);
}
```

### `string.h` usage (examples taken from the Git source tree)

**`memset(dest, value, size)`** — fill a memory area with a given value.

```c
memset(&pool, 0, sizeof(pool));
```

**`memcpy(dest, src, size)`** — copy a memory area into another.

```c
memcpy(map, &blank, sizeof(*map));
```

Prefer `memmove` — it is safe if the areas overlap (at the cost of a temporary area).

**`strcpy` / `strcat` / `strtok` are banned** in the Git repo: unsafe.

**`strstr`** — dynamic pointer positioning. Checks whether a substring is contained in a
main string, returning a pointer on the first occurrence.

```c
const char *eoh;
eoh = strstr(buf->buf, "\n\n");
```

**`strchr`** — dynamic pointer positioning. Checks whether a character is present in a
string, returning a pointer on its first occurrence.

```c
value = strchr(text, '=');
if (value) {
    const char *slash = strchr(url, '/');
}
```

**`strncmp`** — test function. Tells whether two strings are identical over the given
size (returns 0 in that case).

```c
if (!strncmp(prog, "git", 3) && isspace(prog[3]))
```

**`strcmp`** — test function. Tells whether two strings are identical (returns 0).

```c
if (!strcmp(type, blob_type))
```

### Walking a string

```c
const char *original = "The original string.";

// Duplicate the initial string
char *copy = (char *) malloc(strlen(original) + 1);
memcpy(copy, original, sizeof(copy));

// Uppercase every letter
char *ptr = copy;
while (*ptr != '\0') {
    *ptr = toupper(*ptr);
    ptr++;
}

printf("%s\n", copy);

free(copy);
```

---

## Structures: padding, packing, bit fields

### Padding

On x86 the members of a structure are aligned on 4 bytes, so padding is inserted:

```c
struct toto {
    uint8_t a;
    // 3 padding bytes
    uint16_t b;
    // 2 padding bytes
};
```

### Endianness

The first member of a structure is the LSB (bit 0), up to the MSB.

### Decoding example

An I2C frame `request_list` is received as an array of 3 `uint8_t`. Copying those 3 bytes
into a `packed` structure (no padding) gives: first byte into the first member
(`debug_section_id`), second one into `debug_command_id`, third one into the first byte
member of the union.

### Bit fields

```c
struct toto __attribute__((__packed__)) {  // no padding
    uint8_t  a;
    uint16_t b;
    uint8_t  toto : 1;
};
```

Here `toto` holds a single bit.

---

## Function pointers

Function pointer used as a callback:

```c
/* Toto.h */
typedef void (*ci_app_uart_callback_t)(uint32_t event);
```

```c
/* Toto.c */
int32_t mw_initialize_ci_app_uart(const ci_app_uart_callback_t callback,
                                  uint8_t *p_rx_buffer,
                                  const uint32_t buffer_size)
{
}
```

Dispatch table style:

```c
char fn1(char a, char b);

char (*Fct)(char, char);

switch (...) {
    case toto:
        Fct = &fn1;
}
```

---

## External linkage

```c
/* toto.c */
int tata = 0;
```

```c
/* toto.h */
extern int tata;
```

```c
/* kiki.c */
#include "toto.h"    // if you want to use tata
```

---

## Embedded guidelines

NASA's rules (via Low Level Learning):

1. **Simple control flow** — no recursive functions, no `goto`.
2. **Limit all loops** — bound every loop with an upper limit (`MAX_ITER`).
3. **Don't use the heap** — no `malloc`/`free`; avoids memory leaks and use-after-free,
   and keeps the behaviour deterministic.
4. **Limit function size** — 60 lines maximum; good for unit testing.
5. **Data hiding** — make data available at the lowest possible scope, avoid globals.
6. **Check return values** — test every non-void return value.
7. **Limit the preprocessor** — bad for readability, and hard to test because of the
   resulting build configuration explosion.
8. **Restrict pointer use** — only one level of dereferencing, no dereferencing in
   macros, no function pointers.
9. **Be pedantic** — `gcc -Wall -Werror -Wpedantic`.
10. **Unit testing.**

---

## Driver / HAL / CMSIS

From the ST point of view:

- **CMSIS Device Layer** — all data structures specific to an MCU chip.
- **CMSIS Core Layer** — all data structures specific to the core (e.g. CM3).
- **HAL Driver** — in the STM ecosystem the HAL is the software item containing all the
  peripheral drivers (`driver_uart.c`, …).
- **CMSIS Driver** — a HAL superset; don't use.

---

## printf in an embedded system

Example on STM32 — the call chain is:

1. `printf` (stdio)
2. `__io_putchar` overload (user code)
3. `HAL_UART_Transmit()` (STM32 HAL)

```c
int __io_putchar(int ch)
{
    uint8_t c[1];
    c[0] = ch & 0xFF;
    HAL_UART_Transmit(&huart2, &c[0], 1, 10);
    return ch;
}
```

---

## `static inline` benchmark

`make ezairo_cm3 IMAGE_CONFIG=dev_test_config -j`

| Variant | text | data | bss | dec | hex | Time between error & FIFO full |
|---|---|---|---|---|---|---|
| `static inline` in the header | 18972 | 104 | 5976 | 25052 | `61dc` | **160 ns** |
| `.c` / `.h` separated | 19048 | 104 | 5976 | 25128 | `6228` | 1.6 µs |

Inlining costs 76 bytes of `.text` and buys a 10x latency improvement here.

[Back to Languages](./)
