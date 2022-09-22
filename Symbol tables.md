---
date created: 2022-09-21 20:20
---

# Symbol tables

- The main data structure in a compiler after syntax trees.
- Typically dictionary-like
- Main operations:
  - Insert
  - Lookup
  - Delete
- Hash tables are generally the best choice as these main operations can be performed in almost-linear time.

## Hash tables

### Collision resolution

#### Open addressing

We could use open addressing, which could be likened to an array of buckets. Each of these buckets has enough space for a single item, so any colliding items are placed into the next available successive bucket.

Unfortunately, this can lead to poor performance with a poor hash function and it's limited by the size of the bucket array.

#### Separate chaining

Here, each bucket is a linear/linked list, and collisions are resolved by inserting the new item into the list.

### Collision avoidance using hash functions

```ad-question
title: Goal
Convert a character string (the name of the identifier) into an integer in the range `0...size-1` where `size` is the size of the table and of the bucket array.
```

The plan is to convert each character to a non-negative integer, usually using built-in conversion mechanisms of the compiler's implementation language. We then apply an adequate hash function $h$, a good choice for which is typically $h = (\sum_{i=1}^{n} \phi^{n-i}c^i)mod\ size$ where $c_i$ is the numeric value of the $i$th character and $\phi$ is a power of 2 so that the multiplication can be a simple bit-shift.

### Scope

Names have a declaration^[constant declarations, variable declarations, function declarations...]

- Local declarations
  - Names have a limited scope^[visible in a portion of the program]
- Global declarations
  - Names are visible in the entire program

The same name can be declared several times, in which case the use of a name refers to its closest declaration. The **scope** of a declaration is a sub-tree of the syntax tree, meaning that nested declarations give rise to scopes that are nested sub-trees of the syntax tree.

#### Nested scope

One solution is to manage the hash table in a stack-like manner, meaning each declaration of a name in the symbol table contains some kind of reference to the different scopes in which it has been declared. While this works, a better solution may be to build a new symbol table for each scope and to link the tables from inner to outer scopes together.
