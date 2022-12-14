---
date created: 2022-09-21 20:18
date updated: 2022-09-22 08:33
tags:
  - '#non-terminal'
  - '#non-terminals'
---

# Generated Languages

- G = (V, T, S, P)
- L(G) = $\{w\ |\ w \in T^*$ and $S$ $\implies^*\ w\}$
- $T^*$ because $w$ may just be $\epsilon$

## Context-free languages

- L is a context-free language iff there exists a [[Generative Grammars#Context-free grammars free grammars|context-free grammar]] G such that L = L(G)
- $L$ context-free language $\iff \exists$ [[Generative Grammars#Context-free grammars free grammars|context-free grammar]] $G \mid L=L(G)$

### Canonical derivations

- Rightmost (leftmost) derivation step
  - Replace the rightmost (leftmost) #non-terminal
- Canonical derivations of words in the language
  - Either every step is rightmost or every step is leftmost

### Derivation trees

- Start symbol is root
- For every derivation step under the production
  - $A \implies X_1X_2...X_n$
- Generate children $X_1X_2...X_n$ for node $A$
- Terminals^[and [[Conventions#Epsilon epsilon|epsilon]]] are the leaves
- The derived word is read from the leaves of the tree

### Ambiguity

- Grammar $G$ is **ambiguous** iff there exists $w \in L(G)$ that can be generated by two distinct canonical derivations, either both rightmost or both leftmost

#### Arithmetic expressions

- $E \rightarrow E + E \mid E * E \mid n$
- Ambiguous?
  - Take $w = n + n * n$
  - $E \implies E+E \implies n+E \implies n+E*E \implies n+n*E \implies n+n*n$
  - Same applies using solely rightmost derivations

### Properties

#### Closure w.r.t^[with respect to] union

##### Lemma

- The class of free languages ir closed w.r.t set-union, meaning if $L_1$ and $L_2$ are free languages then $L_1 \cup L_2$ is a free language

##### Proof

- Let $L_1$ and $L_2$ be free languages
- Then there exist two free grammars $G_1 = (V_1, T_1, S_1, P_1)$ and $G_2 = (V_2, T_2, S_2, P_2)$ such that $L_1 = L(G_1)$ and $L_2 = L(G_2)$
- Let $V^{'}_{2}$ be a refresh of $V_2$ to avoid possible name clashes with #non-terminals in $V_1$ (no refresh needed when all #non-terminals of $G_2$ are different from those of $G_1$)
- Let $G = (V_1 \cup V^{'}_{2} \cup \{S\},\ T_1 \cup T_2,\ S,\ P_1 \cup P^{'}_{2} \cup \{S \rightarrow S_1 \mid S^{'}_{2}\})$ where
  - $S$ is a new symbol not in $\{V_1 \cup V^{'}_{2}\}$
  - $S^{'}_{2}$ stands for the refresh of $S_2$
  - $P^{'}_{2}$ stands for the refresh of the productions in $P_2$
- Then $L(G)$ is free and $L(G) = L(G_1) \cup L(G_2)$

```ad-note
Symbols from $L_1$ cannot be used in the production of words in $L_2$ and vice-versa. That is, words generated starting with the production $S \rightarrow S_1$ will belong to $L_1$ whereas words generated starting with the production $S \rightarrow S^{'}_{2}$ will belong to $L_2$.
```

###### Why is $G$ free?

- The productions of $G$ are those in $P_1 \cup P^{'}_{2} \cup \{S \rightarrow S_1 \mid S^{'}_{2}\}$
- The productions in both $P_1$ and $P^{'}_{2}$ have the same form as they do before name refreshing, hence the form $A \rightarrow \alpha$
- The productions $S \rightarrow S_1$ and $S \rightarrow S^{'}_{2}$ also have the form $A \rightarrow \alpha$

#### Closure w.r.t concatenation

##### Lemma

- The class of free languages is closed w.r.t concatenation, meaning
  - If $L_1$ and $L_2$ are free languages then $\{w_1w_2 \mid w_1 \in L_1$ and $w_2 \in L_2\}$ is a free language

##### Proof

- Let $L_1$ and $L_2$ be free languages
- Then there exist two free grammars $G_1 = V_1, T_1, S_1, P_1)$ and $G_2 = (V_2, T_2, S_2, P_2)$ such that $L_1 = L(G_1)$ and $L_2 = L(G_2)$
- Notice that, wlog^[without loss of generality], we can assume that there is no clash between the #non-terminals of $G_1$ and those of $G_2$. In fact, we can always apply renaming as done in the proof on closure w.r.t union
- Let $G = (V_1 \cup V_2 \cup \{S\},\ T_1 \cup T_2,\ S,\ P_1 \cup P_2 \cup \{S \rightarrow S_1S_2\})$ where $S$ is a new symbol not in $\{V_1 \cup V_2\}$
- Then $L(G)$ is free and $L(G) = \{w_1w_2 \mid w_1 \in L(G_1)$ and $w_2 \in L(G_2)\}$

#### Closure w.r.t intersection does not hold

##### Lemma

- The class of free languages is not closed w.r.t intersection

##### Proof by contradiction

- Take two free languages $L_1$ and $L_2$ whose intersection is not free
- $L_1 = \{a^nb^nc^j \mid n, j > 0\}$
- $L_2 = \{a^jb^nc^n \mid n > 0\}$
- $L_1 \cap L_2 = \{a^nb^nc^n \mid n > 0\}$

### Exercises

- Define a grammar $G$ such that $L(G)$ is the set of all even binary numbers
  - $S \rightarrow 0S \mid 1S \mid 0$
    - Ensures that the final character/digit is always a zero, as if it were a 1 it would be followed by an S and would therefore not be part of the language
- Define a grammar $G'$ such that $L(G') = \{1^n0 \mid n \ge 0\}$
  - $S \rightarrow A0 \mid 0$
  - $A \rightarrow 1A \mid 1$
- $L(G) = L(G')$?
  - No: $000000 \in L(G)$ but $\notin L(G')$

### Pumping Lemma for [[Generated Languages#Context-free languages|Context-Free Languages]]

#### Lemma

Let $L$ be a free language, then:

- $\exists p \in \mathbb{N}^+$ such that
- $\forall z \in L$ such that $|z| \gt p$
- $\exists u, v, w, x, y$ such that
  - $z = uvwxy$ and
  - $|vwx| \le p$ and
  - $|vx| \gt 0$ and
  - $\forall i \in \mathbb{N}.uv^iwx^iy \in L$

#### Proof

Let $L$ be a free language. The lemma is about words longer than $p \gt 0$, and hence different from $\epsilon$. Then, just consider a "cleaned-up" free grammar $G$ such that $L = L(G)$.

````ad-note
In any derivation tree, every path from the root to a terminal node traverses as many #non-terminal nodes as the length of the path.
```ad-example
The length of a path traversing the nodes $(S, B_1, B_2, ..., B_{k-1}, a)$ is $k$
```
````

Define $p$ to be the length of the longest word that can be derived by derivation trees whose paths from the root are at most as long as the number of #non-terminals in $G$. Let $z \in L$ be such that $|z| \gt p$. Then there is a derivation tree for $z$ which has at least a path whose length is strictly greater than the number of #non-terminals .

Consider the longest path in the tree, and the _deepest pair_ of occurrences of the same #non-terminal along that path where the depth of a pair of #non-terminals is the depth of its second occurrence going bottom-up.

```ad-example
Below the deepest pair is always the pair of $A$s
![[deepest-pair.png]]
```

Let $A$ be the #non-terminal of the deepest pair of occurrences of the same #non-terminal along the path, and call its two occurrences $A1$ and $A2$
![[a1-a2-pair.png]]
Then there are two distinct subtrees with root $A$, i.e., the pink subtree and the green subtree, and $z = uvwxy$:
![[subtrees-rooted-in-a.png]]
We can then say that

- $uv^0wx^0y \in L$
  - ![[v0x0-derivation-tree.png]]
- $uv^2wx^2y \in L$
  - ![[v2x2-derivation-tree.png]]
- $uv^3wx^3y \in L$
  - ![[v3x3-derivation-tree.png]]

It follows that:

- $\forall i \in \mathbb{N}. uv^iwx^iy \in L$
- $|vwx| \le p$
  - Our choice of $(A1, A2)$ guarantees that the depth of the tree rooted at $A2$ is less than the number of #non-terminals and therefore the length of its yield is bound by $p$
- $|vx| \gt 0$
  - As $G$ is cleaned-up, if $A \implies^* \alpha A \beta$ then at least one symbol is derived by either $\alpha$ or $\beta$

#### What can we use it for?

The lemma states that if $L$ is a free language, then the lemma holds. This does not mean that the lemma can be used to show that a given language is free, but it can be used to show that a language is **not** free.

##### Schema

- Assume the language $L$ is free
- Show that the pumping lemma thesis is false, that is prove its inverse
- By contradiction, conclude that $L$ is not free

#### Inverse of the Pumping Lemma Thesis

- $\forall p \in \mathbb{N}^+. \exists z \in L: |z| \gt p. \forall u, v, w, x, y$
  - $(z = uvwxy$ and $|vwx| \le p$ and $|vx| \gt 0) \implies \exists i \in \mathbb{N}.uv^iwx^iy \notin L$

That is:

- For any positive natural number $p$, choose a word $z$ longer than $p$ and belonging to the language.
- Show that, for any unpacking of $z$ into $uvwxy$ with $|vwx| \le p$ and $|vx| \gt 0$ is taken, a natural number $i$ can be chosen which is such that $uv^iwx^iy \notin L$

#### The Pumping Lemma at Work

```ad-example
title: Example 1
Given the grammar $G$:

- $S \rightarrow aSBc \mid abc$
- $cB \rightarrow Bc$
- $bB \rightarrow bb$

$G$ is #context-dependent and $L(G) = \{a^nb^nc^n \mid n \gt 0\}$
Is $L(G)$ a free language?

Suppose $L$ is free, and let $p$ be an arbitrary positive number. Take $z = a^pb^pc^p$.

- If $(z = uvwxy$ and $|vwx| \le p$ and $|vx| \gt 0)$ then
  - $vx$ cannot have occurrences of both $a$s and $c$s because the last occurrence of $a$ and the first occurrence of $c$ are $p+1$ positions apart. In fact, for some positive $k$ and $j$:
    - Either $vwx = a^k$
    - Or $vwx = a^kb^j$
    - Or $vwx = b^j$
    - Or $vwx = b^jc^k$
    - Or $vwx = c^k$

Therefore $vx$ either has no occurrences of $c$ or no occurrences of $a$, meaning $uv^0wx^0y$ cannot have the form $a^nb^nc^n$, hence $uv^0wx^0y \notin L$.
Finally, by contradiction with respect to the pumping lemma, $L$ is not free.
```

````ad-example
Title: Example 2

Given the grammar $G$:

- $S \rightarrow CD$
- $C \rightarrow aCA \mid bCB \mid \epsilon$
- $AD \rightarrow aD$
- $BD \rightarrow bD$
- $Aa \rightarrow aA$
- $Ab \rightarrow bA$
- $Ba \rightarrow aB$
- $Bb \rightarrow bB$
- $D \rightarrow \epsilon$

G is #context-dependent, but what is $L(G)$?

```ad-note
Derived strings have a "bookmark" $D$ initially in the rightmost position: $$S \rightarrow CD$$
```

```ad-note
Strings can only grow longer by replacing $C$: $$C \rightarrow aCA \mid bCB \mid \epsilon$$
```

```ad-note
#Non-terminals close to the rightmost delimiter can be converted to the corresponding terminal: $$AD \rightarrow aD$$ $$BD \rightarrow bD$$
```

```ad-note
#Non-terminals and #terminals can be swapped when the #terminal is to the right of the #non-terminal : $$Aa \rightarrow aA$$ $$Ab \rightarrow bA$$ $$Ba \rightarrow aB$$ $$Bb \rightarrow bB$$ 
```

| Word       | Production Used          |
| ---------- | ------------------------ |
| $S$        | -                        |
| $CD$       | $S \rightarrow CD$       |
| $aCAD$     | $C \rightarrow aCA$      |
| $aaCAAD$   | $C \rightarrow aCA$      |
| $aabCBAAD$ | $C \rightarrow bCB$      |
| $aabBAAD$  | $C \rightarrow \epsilon$ |
| $aabBAaD$  | $AD \rightarrow aD$      |
| $aabBaAD$  | $Aa \rightarrow aA$      |
| $aabaBAD$  | $Ba \rightarrow aB$      |
| $aabaBaD$  | $AD \rightarrow aD$      |
| $aabaaBD$  | $Ba \rightarrow aB$      |
| $aabaabD$  | $BD \rightarrow bD$      |
| $aabaab$   | $D \rightarrow \epsilon$ |

$L(G) = \{ww \mid w \in \{a, b\}^*\}$

```ad-question
title: Is $L(G)$ free?

If we choose $z = a^pb^pa^pb^p$ we can see that, as $|vwx| \le p$, it is impossible to choose $z$ such that $uv^0wx^0y \in L$.
```
````
