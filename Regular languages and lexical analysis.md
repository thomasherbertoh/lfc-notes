---
date created: 2022-09-21 20:20
---

# Regular languages and lexical analysis

## Recognising languages

- $L = \{a^nb^n \mid n > 0\}$ is a free language
- Stack seems ideal to recognise words in $L$
  - Read the string one symbol at a time
  - Push $a$s onto the stack
  - Pop one $a$ for each $b$
  - Reject further $a$s, check emptiness

### Arbitrary strings over the alphabet

- Do we really need all that "counting" to generate arbitrary strings over the alphabet?
- $S \rightarrow a \mid b \mid ... \mid z \mid aS \mid bS \mid ... \mid zS$
- Could just use a [[Finite state automata|state machine]]
- ![[Simple state machine.png]]
- Here, from the start symbol, we go straight to $q_0$
- From $q_0$ we could read either a letter or something else
- If we read a letter
  - We go to $q_1$, from where we can continue to revisit $q_1$ as long as we read letters
  - If we then read something other than a letter, we go to $q_2$
- Otherwise we go straight to $q_2$
- From $q_2$ we can continue to read anything, always coming back to $q_2$. This continues until we run out of things to read.

## Regular languages

- Regular grammars are free grammars with productions of the form
  - $A \rightarrow a$
  - $A \rightarrow aB$
  - $A \rightarrow \epsilon$
- Regular expressions
- (Non)Deterministic finite state automata
- At the basis of lexical analysis

### Closure properties of regular languages

#### Lemma

Regular languages are closed with respect to

- union
- concatenation
- complementation
- intersection

## Regular expressions

- Set an alphabet $A$ and a number of operators
- Define regular expressions inductively
  - Base:
    - Every $a \in A$ is a regular expression
    - $\epsilon$ is a regular expression
  - Step: if $r_1$ and $r_2$ are regular expressions then
    - (Alternation) $r_1 \mid r_2$ is a regular expression
    - (Concatenation) $r_1 \cdot r_2$ is a regular expression (written $r_1r_2$)
    - (Kleene star) $r_{1}^{*}$ is a regular expression
    - (Parentheses) $(r_1)$ is a regular expression
- For order of precedence see [[Conventions#Regular expressions]]

### Denoted language

- Given a regular expression $r$ over $A$, the language denoted by $r$, written $L(r)$, is also inductively defined on the structure of $r$
  - Base:
    - $L(a) = \{a\}$ for every $a \in A$
    - $L(\epsilon) = \{\epsilon\}$
  - Step:
    - If $r = r_1 \mid r_2$ then $L(r) = L(r_1) \cup L(r_2)$
    - If $r = r_1r_2$ then $L(r) = \{w_1w_2 \mid w_1 \in L(r_1)$ and $w_2 \in L(r_2)\}$
    - If $r = r_{1}^{*}$ then $L(r) = \{\epsilon\} \cup \{w_1w_2...w_k \mid k \ge 1$ and $\forall i : 1 \le i \le k,\ w_i \in L(r_1)\}$
    - If $r = (r_1)$ then $L(r) = L(r_1)$

### Examples

- $L(a \mid b) = \{a, b\}$
- $L((a \mid b)(a \mid b)) = \{aa, ab, ba, bb\}$
- $L(a^*) = \{a^n \mid n \ge 0\}$
- $L(a \mid a^*b) = \{a\} \cup \{a^nb \mid n \ge 0\}$
- $(a \mid b \mid ... \mid z)(a \mid b \mid ... \mid z)^*$ denotes the set of words over the alphabet
- $(0 \mid 1)^*0$ denotes all the even binary numbers
- $b^*(abb^*)^*(a \mid \epsilon)$ denotes the set of words over $\{a, b\}$ with no consecutive occurrences of $a$
- $(a \mid b)^*aa(a \mid b)^*$ denotes the set of words over $\{a, b\}$ with consecutive occurrences of $a$

## Lexical analysis

Provides input to syntax analysis

- Typical choice of tokens:
  - One token for each keyword
  - Tokens for operators (or for classes of operators)
  - One token for identifiers
  - Tokens for punctuation symbols
- Task:
  - Recognise [[Regular languages and lexical analysis#Lexemes|lexemes]]
  - Return tokens (pairs of token-name and token-value)

### Lexemes

- Described by [[Regular languages and lexical analysis#Regular expressions|regular expressions]]
- Recognised by a state machine that can take appropriate actions when recognising words
