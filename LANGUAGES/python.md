---
layout: default
title: Python
---

# Python

- [Lists and tuples](#lists-and-tuples)
- [Dictionaries](#dictionaries)
- [Formatting & slices](#formatting--slices)
- [Iteration idioms](#iteration-idioms)
- [Virtual environments](#virtual-environments)
- [NumPy](#numpy)
- [Misc](#misc)
- [DUNOD puzzle book — key concepts](#dunod-puzzle-book--key-concepts)

---

## Lists and tuples

```python
list1 = list()
list2 = []
```

Member functions: `append`, `pop(index)`, `insert(index, value)`, `extend`,
`remove(value)`, `index(value)`, `sort`, `count` (occurrences of the argument),
`in` (True if the argument appears), `sum`, `copy`.

- **Lists are mutable.**
- **Strings are immutable.**

> **Never modify something while iterating over it:**
> `for number in numbers.copy():`

---

## Dictionaries

```python
for key, value in dico1.items():
    print(f"Couple cle {key}, et valeur {value}")

dictionnaire[cle] = valeur  # adds or replaces the value of a key
```

---

## Formatting & slices

```python
print(f"toto = {toto}")
```

Very handy with slices:

```python
toto[:12]   # the first 12 elements
toto[13:]   # from the 13th element to the end of the array
```

---

## Iteration idioms

```python
for index, chiffre in enumerate(list1):
    print(f"Indice {index}: element {chiffre}")

[print(nb * nb) for nb in list1]
```

---

## Virtual environments

```powershell
python -m venv venv
.\venv\Scripts\activate
```

---

## NumPy

Smart way to build lists without loops.

```python
x = np.zeros(10)
x = np.ones(10)
x = np.linspace(2, 10, 5)
x = np.array([10, 20])

np.min(x)
np.max(x)
np.argmin(x)    # index of the min value
np.argmax(x)    # index of the max value
x[-1]           # last element
np.sort(x)

mask = np.where(toto > 0, 10, -10)  # if toto is positive replace with 10, else -10

a_array @ b_array                   # matrix multiplication

a = np.array([1, 2, 3], dtype='int16')
a = x.reshape((4, 2))

np.genfromtxt('data.txt', delimiter=',')
```

---

## Misc

Pipe a Python result into the stdin of another program — here an ELF executable:

```bash
python3 -c "print('aaaa\x00aaaa\x00')" | ./main
```

---

## DUNOD puzzle book — key concepts

**Puzzle 1 — the `in` keyword in `if` and `for`:**

```python
for i in range(10):     # i goes from 0 to 9 in steps of 1
for x in (-1, 0, 1):    # iterate over the values of a tuple
if y in ['a', 'z']:     # test whether a variable belongs to a given list
```

**Puzzle 2 — Reverse Polish notation calculator:** work as a *stack* over the list of
characters.

**Puzzle 3 — polynomials:**

- Degree 1 → a straight line (at least two points are needed for the plot).
- Degree 4 → 5 points are needed: $C_0 + C_1x + C_2x^2 + C_3x^3 + C_4x^4$.

**Puzzle 4 — hash map:** key is the hash, value is the password. Alternative: write the
data dictionary into a JSON file.

**Puzzle 6 — shortest path (Dijkstra):**

- Look at the neighbours of the current city.
- Accumulate the distance from point A (the start) with the current city → neighbour
  interval, and populate a dictionary with the distance information between each vertex
  and the starting point.
- Put the next cities in a list; on the following iteration sort the list and only keep
  the city with the shortest distance.
- Goal: populate a dict with the cumulated distances between each vertex and point A.

**Puzzle 7 — Huffman coding with a binary tree:**

- Build the tree using objects and recursion.
- Build it starting from the leaves and going up to the root.
- To determine the character codes, go from the root down to the leaves.

**Puzzle 8 — bit string decoding:**

- Take a string representing bits, slice it into blocks.
- Convert it to `int` in both byte orders (big and little).
- Then to a Unicode character with `chr()`.
- Part b: `map()` lets you cast the values of a container.

[Back to Languages](./)
