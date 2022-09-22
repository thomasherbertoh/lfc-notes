---
date created: 2022-09-21 20:18
---

# Conventions

## Epsilon ($\epsilon$)

- Special char used to denote the empty word
- Length of $\epsilon$ = 0
- $\epsilon \equiv \epsilon \epsilon$
- $\epsilon \equiv b^0$ for every terminal $b$

## Regular expressions

- Kleene star has highest precedence, is left associative
- Concatenation has second highest precedence, is left associative
- Alternation has lowest precedence, is left associative
- => $(a \mid bc^*)$ means $(a \mid b(c^*))$
- which means $(a \mid (b(c^*)))$
