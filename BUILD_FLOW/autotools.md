---
layout: default
title: Autotools
---

# Autotools

## Simplified flow

![Simplified Autotools flow: configure.ac to Makefile via autoconf, automake and configure](img/autotools-flow.png)

## Full pipeline

![Autotools full pipeline: autoscan, configure.ac, aclocal, autoheader, automake, autoconf, config.status, Makefile](img/autotools-flow-detailed.png)

`configure.ac` and `Makefile.am` are the input files. `autoconf` turns `configure.ac`
into the `configure` shell script; `automake` turns `Makefile.am` into `Makefile.in`.
Running `./configure` then produces the final `Makefile`, which `make` uses to build the
target.

[Back to Build Flow](./)
