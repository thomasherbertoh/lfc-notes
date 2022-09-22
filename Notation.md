---
date created: 2022-09-21 20:19
tags:
  - '#non-terminals'
date updated: 2022-09-21 20:19
---

# Notation

- Capital letters for #non-terminals
- Uppercase, early in the alphabet
  - $A, B, ... \in (V \setminus T)$ (in set of #non-terminals )
- Uppercase, late in the alphabet
  - $X, Y, ... \in V$
- Lowercase, early in the alphabet
  - $a, b, ... \in T$
- Lowercase, early in Greek alphabet
  - $\alpha, \beta, ... \in V^*$
- $V^*$ means "zero or more repetitions of elements of V"
- $V^+$ means "one or more repetitions of elements of V"
- Strings of terminals
  - $w, w_0, w_1, ...$
- $\{w | w \in \{0, 1\}^*$ and $\#(0, w) = \#(1, w)\}$ means "w such that w is in the set made up of zero or more elements of the set {0, 1} and the number of occurrences of 0 in the word w is equal to the number of occurrences of 1 in the word w"
