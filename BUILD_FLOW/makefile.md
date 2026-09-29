---
layout: default
title: Makefile
---

# Makefile

```makefile
all: toto_app

CC = gcc
INCLUDE = ./include
CFLAGS = -g -Wall -ansi

toto_app: main.o 2.o 3.o
	$(CC) -o toto_app main.o 2.o 3.o
main.o: main.c include/a.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c main.c
2.o: 2.c include/a.h include/b.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 2.c
3.o: 3.c include/b.h include/c.h
	$(CC) -I$(INCLUDE) $(CFLAGS) -c 3.c

clean:
	-rm *.o toto_app
```

## Principle

Describe recipes: define targets and their dependencies.

For a target that does not depend on any file, add it to `.PHONY` (e.g. `clean`).

[Back to Build Flow](./)
