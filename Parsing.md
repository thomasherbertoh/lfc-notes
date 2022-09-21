# Parsing
Given a grammar $G = (V, T, S, P)$ and a word $w$, say whether $w \in L(G)$ and, if so, provide its derivation tree.
- Two relevant kinds of parsing
	- [[Parsing#Top-down parsing|Top-down parsing]]: construct leftmost derivation from the root to the yield
	- [[Parsing#Bottom-up parsing|Bottom-up parsing]]: construct rightmost derivation (in reverse order) from the yield to the root

## Top-down parsing
```ad-example
title: Example 1
Let $w = bd$ and
$G:$
$S \rightarrow Ad \mid Bd$
$A \rightarrow a$
$B \rightarrow b$

A leftmost derivation of $w$ is $S \implies Bd \implies bd$
```
````ad-example
title: Example 2
Let $w = id + id * id$ and
$G:$
$E \rightarrow TE'$
$E' \rightarrow +TE' \mid \epsilon$
$T \rightarrow FT'$
$T' \rightarrow *FT' \mid \epsilon$
$F \rightarrow (E) \mid id$

What is, if one exists, a leftmost derivation of $w$ in $G$?
$E \implies TE' \implies FT'E' \implies (E)T'E' \implies TE'T'E' \implies FT'E'T'E' \implies idT'E'T'E' \implies idE'T'E' \implies id+TE'T'E' \implies id+FT'E'T'E' \implies id+idT'E'T'E' \implies id+id*FT'E'T'E' \implies id+id*idT'E'T'E' \implies id+id*id$

```ad-note
This involved a large amount of luck, as at several points we could have taken a different path and not been able to achieve the desired result.
```
````

### Predictive top-down parsing
- No backtrack
- $LL(1)$ grammars

#### Top-down parsing table for example 2
| Symbol | id                  | +                         | *                     | (                   | )                         | $                         |
| ------ | ------------------- | ------------------------- | --------------------- | ------------------- | ------------------------- | ------------------------- |
| $E$    | $E \rightarrow TE'$ |                           |                       | $E \rightarrow TE'$ |                           |                           |
| $E'$   |                     | $E' \rightarrow +TE'$     |                       |                     | $E' \rightarrow \epsilon$ | $E' \rightarrow \epsilon$ |
| $T$    | $T \rightarrow FT'$ |                           |                       | $T \rightarrow FT'$ |                           |                           |
| $T'$   |                     | $T' \rightarrow \epsilon$ | $T' \rightarrow *FT'$ |                     | $T' \rightarrow \epsilon$ | $T' \rightarrow \epsilon$ |
| $F$    | $F \rightarrow id$ |                           |                       | $F \rightarrow (E)$ |                           |                           | 

#### Algorithm for predictive top-down parsing
```ad-info
title: Input
String $w$
Top-down parsing table $M$ for $G = (V, T, S, P)$
```
```ad-info
title: Output
Leftmost derivation of $w$ if $w \in L(G)$, `error()` otherwise
```
```ad-info
title: Initialisation
$w\$$ in the input buffer
$\$S$ onto the stack, with $S$ on the top
```
```python
let b be the first symbol of w$
let X be the top of the stack
while X != $ do
	if X == b then
		pop X
		let b be the next symbol of w$
	else if X is a terminal then error()
	else if M[X, b] is error then error()
	else if M[X, b] == X -> Y1...Yk then
		output(X -> Y1...Yk)
		pop X
		push Yk
		...
		push Y1
	let X be the top of the stack
```
```ad-example
title: Example for $w = id + id * id$
| stack      | input            | output                    |
| ---------- | ---------------- | ------------------------- |
| $\$E$      | $id + id * id\$$ | $E \rightarrow TE'$       |
| $\$E'T$    | $id + id * id\$$ | $T \rightarrow FT'$       |
| $\$E'T'F$  | $id + id * id\$$ | $F \rightarrow id$        |
| $\$E'T'id$ | $id + id * id\$$ |                           |
| $\$E'T'$   | $+ id * id\$$    | $T' \rightarrow \epsilon$ |
| $\$E'$     | $+ id * id\$$    | $E' \rightarrow +TE'$     |
| $\$E'T+$   | $+ id * id\$$    |                           |
| $\$E'T$    | $id * id\$$      | $T \rightarrow FT'$       |
| $\$E'T'F$  | $id * id\$$      | $F \rightarrow id$        |
| $\$E'T'id$ | $id * id\$$      |                           |
| $\$E'T'$   | $*id\$$          | $T' \rightarrow *FT'$     |
| $\$E'T'F*$ | $*id\$$          |                           |
| $\$E'T'F$  | $id\$$           | $F \rightarrow id$        |
| $\$E'T'id$ | $id\$$           |                           |
| $\$E'T'$   | $\$$             | $T' \rightarrow \epsilon$ |
| $\$E'$     | $\$$             | $E' \rightarrow \epsilon$ |
| $\$$       | $\$$             |                           | 
```
How do we fill the entries of parsing tables?
- The entry $M[A, b]$ is consulted to expand $A$ when the next input character is $b$
- Then we set $M[A, b] = A \rightarrow \alpha$ if
	- Either $\alpha \implies^* b\beta$
	- Or $\alpha \implies^* \epsilon$ and we can have $S \implies^* wA\gamma$ with $\gamma \implies^* b\beta$

#### $first(\alpha)$
Set of terminals that begin strings derived from $\alpha$
Also, if $\alpha \implies^* \epsilon$ then $\epsilon \in first(\alpha)$
$first(\epsilon) = \{\epsilon\}$
$first(a) = \{a\}$
$first(A) = \bigcup_{A \rightarrow \alpha} first(\alpha)$
Computation of $first(Y_1, ..., Y_n)$:
```python
first(Y1, ..., Yn) = empty_set
j = 1
while j <= n do
	add first(Yj) \ {epsilon} to first(Y1, ..., Yn)
	if epsilon in first(Yj) then j++
	else break
if j == n + 1 then add epsilon to first(Y1, ..., Yn)
```

#### $follow(A)$
$follow(A)$ is the set of terminals that can follow $A$ in some derivation
```ad-example
title: Algorithm for $follow(A)$
~~~python
follow(S) = {$}
for A != S
	follow(A) = empty_set
while not saturated
	for B -> aAb
		if b != epsilon
			add first(b) \ {epsilon} to follow(A)
		if b == epsilon or epsilon in first(b)
			add follow(B) to follow(A)
~~~
```

### Construction of predictive parsing tables
```ad-example
title: Algorithm
input:	Grammar G = (V, T, S, P)
output:	Predictive parsing table M

~~~python
for (A -> a) in P:
	add (A -> a) to M[A, b] for b in first(a)
	if epsilon in first(a):
		add (A -> a) to M[A, x] for x in follow(A)
set all empty entries to error()
~~~
```
```ad-note
title: Recall
By [[Notation|notational convention]] $b$ is a terminal symbol
```
```ad-attention
title: Observe
$follow(A)$ can contain $\$$, hence we use $x$ (rather than $b$) to range over it
```

#### $LL(1)$ grammars
If no entry of the predictive parsing table for $G$ is multiply-defined then $G$ is an $LL(1)$ grammar
````ad-example
$E \rightarrow E + T \mid T$
$T \rightarrow T * F \mid F$
$F \rightarrow (E) \mid id$

Is it $LL(1)$?
$first(E) = first(T) = first(F) = \{(, id\}$

Hence, for example, $M[E, id]$ contains both $E \rightarrow E + T$ and $E \rightarrow T$

```ad-question
title: What if we always expand $E$ into $E + T$ when we see the input $id$?
Infinite sequence of $E + T + T + T + ...$
```
````

#### Left recursion
A grammar is **left recursive** if, for some $A$ and some $a$, $A \implies^* Aa$
```ad-example
$S \rightarrow B \mid a$
$B \rightarrow Sa \mid b$

Left recursive by $S \implies B \implies Sa$
```

A grammar is **immediately left recursive** if it has a production of the form $A \rightarrow Aa$
```ad-example
$E \rightarrow E + T \mid T$
$T \rightarrow T * F \mid F$
$F \rightarrow (E) \mid id$

Immediately left recursive by either $E \rightarrow E + T$ or $T \rightarrow T * F$
```

```ad-abstract
title: Lemma
If $G$ is left recursive, then $G$ is not $LL(1)$
```

##### Elimination of immediate left recursion
We want to transform $A \rightarrow Aa \mid \beta$ where $a \neq \epsilon$ and $\beta \neq A \gamma$ to get a non left recursive grammar which generates the same language
```ad-abstract
title: Strategy
Substitute $A \rightarrow Aa \mid \beta$ where $a \neq \epsilon$ and $\beta \neq A \gamma$ with
$A \rightarrow \beta A^{'}$
$A^{'} \rightarrow aA^{'} \mid \epsilon$
where $A^{'}$ is a fresh non-terminal
```
```ad-note
This basically inverts the tree representation of the grammar (see below). Instead of using $\beta$ as the ending point for the recursion, it's now used to start everything off.
```
```ad-note
We take any non-left-recursive productions of $A$ and place them before a fresh non-terminal ($A^{'}$), whose productions will then contain the original left-recursive productions along with $\epsilon$ to ensure that the recursion can end, as without it all productions contain a non-terminal.
```

###### Tree representation of elimination of immediate left recursion
A representation of the example grammar could be the following
![[Tree representation of grammar with immediate left recursion.png]]
This can easily be modified to give
![[Tree representation of grammar removed immediate left recursion.png]]

###### Generalisation
For the most general case, substitute $A \rightarrow Aa_1 \mid ... \mid Aa_n \mid \beta_1 \mid ... \mid \beta_k$ where $a_j \neq \epsilon$ for every $j = 1...n$ and $\beta_i \neq A\gamma_i$ for every $i = 1...k$ with
$A \rightarrow \beta_1A^{'} \mid ... \mid \beta_kA^{'}$
$A^{'} \rightarrow a_1A^{'} \mid ... \mid a_nA^{'} \mid \epsilon$
where $A^{'}$ is a fresh non-terminal.

##### Elimination of left recursion
###### Intuition
- Transform the grammar so as to decrease the number of steps of the derivation $A \implies^* Aa$
- Eliminate immediate left recursion

````ad-example
Take
$A \rightarrow Ba \mid b$
$B \rightarrow Bc \mid Ad \mid b$
Left recursion shows in two dervation steps: $$A \implies Ba \implies Ada$$
To decrease the number of steps of the derivation, replace the production $B \rightarrow Ad$ with $$B \rightarrow Bad \mid bd$$
We get
$A \rightarrow Ba \mid b$
$B \rightarrow Bc \mid Bad \mid bd \mid b$

```ad-note
We have basically replaced the $A$ from the production $B \rightarrow Ad$ with all of it's productions.
```

If we eliminate immediate left recursion from $B$ we get the grammar
$A \rightarrow Ba \mid b$
$B \rightarrow bdB^{'} \mid bB^{'}$
$B^{'} \rightarrow cB^{'} \mid adB^{'} \mid \epsilon$
````

```ad-note
Eliminating left recursion doesn't guarantee an $LL(1)$ grammar
```
```ad-note
Eliminating left recursion doesn't eliminate ambiguity
```
```ad-summary
No left-recursive grammar is $LL(1)$
```

#### Left factoring
Take the grammar $S\rightarrow aSb \mid ab$
```ad-question
title: Is it [[Parsing#LL 1 grammars|LL(1)]]?
No; the entry $[S, a]$ of the parsing table contains two productions:
$S \rightarrow aSb$
$S \rightarrow ab$
```

A grammar can be **left factorised** if multiple productions for the same non-terminal have the same prefix
```ad-abstract
title: Lemma
If $G$ can be left factorised then $G$ is not $LL(1)$
```
```ad-abstract
title: Strategy
Delay the choice between productions with the same prefix as much as possible
```
Substitute $A \rightarrow a\beta_1 \mid a\beta_2$ with
$A \rightarrow aA^{'}$
$A^{'} \rightarrow \beta_1 \mid \beta_2$
where $A^{'}$ is a fresh non-terminal
```ad-note
Left factorisation doesn't guarantee an $LL(1)$ grammar
```
```ad-note
Left factorisation doesn't eliminate ambiguity
```
```ad-summary
No grammar that can be left-factorised is $LL(1)$
```

#### Dangling else
##### Innermost binding
A common strategy to avoid amiguity due to a dangling else is to impose the so-called **innermost binding**: every `else` must match with the closest unmatched `then`. Innermost binding can be enforced by defining a grammar that allows only matched `then-else` pairs between occurrences of `then` and `else`
```ad-example
$S \rightarrow M \mid U$
$M \rightarrow$ `if b then` $M$ `else` $M \mid c$
$U \rightarrow$ `if b then` $S \mid$ `if b then` $M$ `else` $U$
```
```ad-summary
No ambiguous grammar is $LL(1)$
```

## Bottom-up parsing
```ad-abstract
title: Definition
Reconstruct, in reverse order, the rightmost derivation of a word from the yield of the tree to its root
```
Various techniques are available ^[e.g. LR(1), LALR(1), SLR(1)], but all bottom-up parsing techniques share fundamental elements:
- $P$ can always be extended to $P^{'}$ by adding the production $S^{'} \rightarrow S$ where $S^{'}$ is a new non-terminal
- Same shift/reduce algorithm for parsing
- Characteristic automata are always used as controllers of the parsing algorithm

Depending on the technique, characteristic automata encode finer or coarser information. The finer the information, the bigger the characteristic automata, the more powerful the parsing technique, the bigger the set of analysed grammars.

### Characteristic automata
States are sets of items
- Items:
	- LR(0) items - $A \rightarrow \alpha \cdotp \beta$
	- LR(1) items - $[A \rightarrow \alpha \cdotp \beta, \Delta]$ where $L \subseteq T \cup \{\$\}$
- Characteristic automata
	- LR(0)-automata - states are sets of [[Parsing#LR 0 -items|LR(0)-items]]
	- LR(1)-automata - states are sets of [[Parsing#LR 1 -items|LR(1)-items]]

#### LR(0)-items
Consider the LR(0)-item $S^{'} \rightarrow \cdot S$
It intuitively means that we have not yet seen any symbol of the word that we are going to parse, and parsing will be successful only if the word we analyse derives from $S$. Therefore, the item $S^{'} \rightarrow \cdot S$ is surely in the initial state of the characteristic automaton, say $P_0$^[$P_0$ should also contain other items].

Consider the grammar $S \rightarrow aSb \mid ab$. If we start analysing a word and expect to see either a derivation from $aSb$ or a derivation from $ab$, then the LR(0)-items $S \rightarrow \cdot aSb$ and $S \rightarrow \cdot ab$ should also be in $P_0$.

##### Closure of sets of LR(0)-items
Let $P$ be a set of LR(0)-items. $closure_0(P)$ is the smallest set of items that satisfies the following equation $$closure_0(P) = P \cup \{B \rightarrow \cdot\ \gamma \mid A \rightarrow \alpha \cdot \beta \in closure_0(P) \land B \rightarrow \gamma \in P^{'}\}$$
Fixed point computation: we can initialise $closure_0(P)$ as $P$, and then add items as needed until we reach the fixed point.
````ad-example
Given
$E \rightarrow E + T \mid T$
$T \rightarrow T * F \mid F$
$F \rightarrow (E) \mid id$
compute $closure_0(\{E^{'} \rightarrow \cdot E\})$
Init: $closure_0(\{E^{'} \rightarrow \cdot E\}) = \{E^{'} \rightarrow \cdot E\}$
Add $E \rightarrow \cdot E + T$ and $E \rightarrow \cdot T$
Add $T \rightarrow \cdot T * F$ and $T \rightarrow \cdot F$
Add $F \rightarrow \cdot (E)$ and $F \rightarrow \cdot id$
```ad-note
To do this, we first calculated the production $E^{'} \rightarrow \cdot E$. Then, seeing as in $P$ there is a non-terminal preceded by a dot ^[in this case $E$], we add all the productions obtained from that non-terminal. This continues until there are either no longer any unvisited non-terminals preceded by a dot in $P$ or there are no more productions that haven't been added yet.
````

##### Construction of LR(0)-automaton
Construct the automaton by populating a collection of states while defining the transition function $P_0 = closure_0(\{S^{'} \rightarrow \cdot S\})$. If an already collected state $P$ contains an item of the form $A \rightarrow \alpha \cdot Y \beta$ then there is a transition from $P$ to a state $Q$ which contains the item $A \rightarrow \alpha \cdot Y \beta$ and, since $Q$ contains $A \rightarrow \alpha \cdot Y \beta$, it also contains all the items in $closure_0(\{A \rightarrow \alpha \cdot Y \beta\})$
```ad-example
Given
$S \rightarrow aABe$
$A \rightarrow Abc \mid b$
$B \rightarrow d$

State 0:
$S^{'} \rightarrow \cdot S$
$S \rightarrow \cdot aABe$^[we have to add the productions of $S$]
Seeing as there are no more productions with markers preceding non-terminal characters, state 0 is finished.

As they are not already there, we have to add two more states to the collection:
$\tau(0, S)$ ^[$\tau($state of provenance, transition of provenance$)$]
$\tau(0, a)$

$\tau(0, S)$ (state 1) has no transitions from its kernel $S^{'} \rightarrow S \cdot$ ^[to obtain the kernel, we move the marker to be after the character that brought us here, in this case $S$] so there are no more states to be added to the collection at this stage.

We now consider $\tau(0, a)$ which is state 2. Seeing as the kernel ^[$S \rightarrow a \cdot ABe$] gives us at least one production with a non-terminal to the right of the marker, we have to add it's closure to this state, that being $\{A \rightarrow \cdot ABc, A \rightarrow \cdot b\}$

Therefore, state 2 is defined as
$S \rightarrow a \cdot ABe$
$A \rightarrow \cdot Abc$
$A \rightarrow \cdot b$

We can now add two more states, those being $\tau(2, A)$ and $\tau(2, b)$.

From $\tau(2, A)$ (state 3), we obtain the productions $S \rightarrow aA \cdot Be$ and $A \rightarrow A \cdot bc$. Seeing as the first of these has $B$ preceded by the marker, we add $B \rightarrow \cdot d$ to state 3.

We should now add $\tau(3, B)$, $\tau(3, b)$, and $\tau(3, d)$ to our collection.

After this, state 3 can be defined as
$S \rightarrow aA \cdot Be$
$A \rightarrow A \cdot bc$
$B \rightarrow \cdot d$

From $\tau(2, b)$ (state 4), we obtain $A \rightarrow b \cdot$, from which there are no transitions.

Let's now consider $\tau(3, B)$ (state 5). We obtain the single production $S \rightarrow aAB \cdot e$. We can add the state $\tau(5, e)$ to our collection, and state 5 is defined as
$S \rightarrow aAB \cdot e$

Considering $\tau(3, b)$ (state 6), we again obtain a single production, this time $A \rightarrow Ab \cdot c$. We can add the state $\tau(6, c)$ to our collection and define state 6 as
$A \rightarrow Ab \cdot c$

Considering the final transition from state 3 ($\tau(3, d)$), which gives us state 7, we get the single production $B \rightarrow d \cdot$ from which there are no more transitions, so state 7 is defined as
$B \rightarrow d \cdot$

We now move on to consider $\tau(5, e)$ (state 8). This again gives us a single production, this time $S \rightarrow aABe \cdot$, from which there are no more transitions and therefore we define state 8 as
$S \rightarrow aABe \cdot$

The same is true for $\tau(6, c)$ (state 9) which gives us $A \rightarrow Abc \cdot$. There are no transitions from here so we define state 9 as
$A \rightarrow Abc \cdot$

From this we can construct the following automaton
![[LR(0)-automaton.png]]
```

#### Shift/reduce parsing tables
The moves of the shift/reduce algorithm are driven by a parsing table, which has one row for each state of the chosen characteristic automaton, and one column for each symbol in $V \cup \{\$\}$
- Shift moves depend on the transition function of the automaton
- Reduction of the production $A \rightarrow \beta$ is done when the parser is in a state containing the reducing item for $A \rightarrow \beta$ ^[Item $A \rightarrow \beta \cdot$ in the case of LR(0)-automata, item $[A \rightarrow \beta \cdot, \Delta]$ in the case of LR(1)-automata]
	- Reductions depend on the **lookahead function** $LA$ which returns a subset of $V \cup \{\$\}$ for each pair $(P, A \rightarrow \beta)$ such that $P$ contains the reducing item for $A \rightarrow \beta$
	- Another way of looking at reductions is this:
		- If, in our word, we find a sequence of characters which is the body of one of the productions of our grammar, we can reduce this sequence of characters to the driver of the production.
		- More formally; if in our word we find a sequence of characters $w^{*}_{0}\beta w^{*}_{1}$, where $w_0$ and $w_1$ are not necessarily different and $\beta$ can be made up of multiple characters and there exists a production in our grammar $A \rightarrow \beta$ where $A$ is any non-terminal, we can replace $\beta$ with $A$.

Distinct choices of characteristic automaton and of lookahead functions drive the construction of parsing tables for distinct techniques of bottom-up parsing (e.g., SLR(1), LR(1), LALR(1)). The class of analysable grammars and the size of parsing tables both depend on this choice, but the procedure to fill in the table and the parsing algorithm is always the same.

```ad-abstract
title: Construction
Fill in each entry $(P, Y)$ according to the following rules:
- Insert "Shift $Q$" if $Y$ is a terminal and $\tau(P, Y) = Q$
- Insert "Reduce $A \rightarrow \beta$" if $P$ contains the reducing item for $A \rightarrow \beta$ and $Y \in LA(P, A \rightarrow \beta)$
- Set to "Accept" if $P$ contains the **accepting item** and $Y = \$$
	- Item $S^{'} \rightarrow S \cdot$ in the case of LR(0)-automata
	- Item $[S^{'} \rightarrow S \cdot, \Delta]$ in the case of LR(1)-automata
- Set to "Goto Q" if $Y$ is a non-terminal and $\tau(P, Y) = Q$
```

##### Conflicts
The table can have multiply-defined entries
- s/r conflict (shift/reduce conflict) - if the entry contains both a shift and a reduce move
- r/r conflict (reduce/reduce conflict) - if the entry contains two reduce moves for distinct productions

#### SLR(1) parsing tables
SLR(1) parsing tables are obtained by taking:
- Characteristic automaton (LR(0)-automaton)
- Lookahead function $LA(P, A \rightarrow \beta) = follow(A)$ for every $A \rightarrow \beta \cdot \in P$

A grammar $G$ is SLR(1) iff its SLR(1) parsing table has no conflicts.

````ad-example
Construct the SLR(1) parsing table for
$S \rightarrow aABe$
$A \rightarrow Abc \mid b$
$B \rightarrow d$

The LR(0)-automaton for the above grammar could look like this
![[LR(0)-automaton for SLR(1).png]]
where states containing reducing items are drawn as final states, and states containing the accepting item are coloured in green.
4 = $\{A \rightarrow b \cdot\}$
7 = $\{B \rightarrow d \cdot\}$
8 = $\{S \rightarrow aABe \cdot\}$
9 = $\{A \rightarrow Abc \cdot\}$

With reference to this automaton, if $Y$ is a terminal and we are given a transition $\tau(P, Y) = Q$, or rather a transition that passes from the state $P$ to the state $Q$ through an edge labelled $Y$, we add a "shift $Q$" operation to the cell $M[P, Y]$ in the parsing table.
```ad-example
If we were given $\tau(0, a) = 2$, we would insert $M[0, a] = shift 2$ into the parsing table.
```
In the case of cells of the form $M[P, Y]$, where $Y$ is a non-terminal and for which there exists a transition $\tau(P, Y) = Q$ we insert a "goto $Q$" operation.
```ad-example
In the cell $M[0, S]$ we would insert "goto 1", as $\exists \tau(0, S) = 1$
```
````

##### Construction of a bottom-up parsing table
Here is the bottom-up parsing table for the above automaton.

|     | a   | b   | c   | d   | e   | $   | S   | A   | B   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0   | s2  |     |     |     |     |     | 1   |     |     |
| 1   |     |     |     |     |     | Acc |     |     |     |
| 2   |     | s4  |     |     |     |     |     | 3   |     |
| 3   |     | s6  |     | s7  |     |     |     |     | 5   |
| 4   |     | r3  |     | r3  |     |     |     |     |     |
| 5   |     |     |     |     | s8  |     |     |     |     |
| 6   |     |     | s9  |     |     |     |     |     |     |
| 7   |     |     |     |     | r4  |     |     |     |     |
| 8   |     |     |     |     |     | r1  |     |     |     |
| 9   |     | r2  |     | r2  |     |     |     |     |     |

From state 0, if we read $a$ we go to state 2, so we write s2^[shift 2] in cell $M[0, a]$. On the other hand, if we were to read $S$ we would move directly into state 1. Seeing as $S$ is a non-terminal, this is a $goto\ 1$ operation which I'll simplify to just 1.

From state 1, nothing happens unless we read the end-of-word character $\$$^[a.k.a the accepting item], in which case we write "Acc" for "$accept$" in cell $M[1, \$]$.

From state 4 it appears as though we can't do anything, but as we have just written $b$, this can be reduced to $A$ as there is a production $A \rightarrow b$ in our grammar. This is the 3rd production in our grammar so we'll write $r3$. To figure out in which cell(s) to write $r3$, though, we have to look at the set $follow(A)$. In this case, the only characters that can follow $A$ are $b$ and $d$, so we write $r3$ into cells $M[4, b]$ and $M[4, d]$.

The rest of the steps involved in creating this parsing table are analogous to those cited above.

```ad-note
As there are no conflicts in the parsing table, the grammar is considered to be SLR(1).
```

##### Shift/reduce parsing
```ad-example
title: Algorithm
Input: String $w$, shift/reduce parsing table $M$ for $G = (V, T, S, P)$
Output: Rightmost derivation of $w$ in reverse order if $w \in L(G)$, $error()$ otherwise
Initialisation: $P_0$ onto the state-stack $stSt$; nothing onto the symbol-stack $symSt$; $w\$$ in the input buffer
```

Let's apply the algorithm to the parsing table obtained previously.
```ad-example
Given the string $abbcde\$$, we'll obtain the derivation steps taken to construct it using the parsing table we made earlier.

We start in state 0^[$stSt = \{0\}$] and we're reading $a$, so we move into state 2.
$stSt = \{0, 2\}$
$symSt = \{a\}$

From state 2, we then read a $b$, which takes us to state 4.
$stSt = \{0, 2, 4\}$
$symSt = \{a, b\}$

We're now in state 4 and we have to read another $b$. The parsing table tells us that this is a reduce operation, and we need to use production 3^[$A \rightarrow b$]. As the length of the body of production 3 is just 1, we remove one item from the state stack and the symbol stack. We add to the symbol stack the driver of production 3^[$A$] and we add to the state stack the state we reach if we read $A$ from the state which is at the top of the state stack, in this case we travel from 2 to 3.
$stSt = \{0, 2, 3\}$
$symSt = \{a, A\}$

In the previous step, we never actually read the second $b$ of the string; it just prompted us to perform a reduce operation. We now read that second $b$ from state 3, taking us to state 6.
$stSt = \{0, 2, 3, 6\}$
$symSt = \{a, A, b\}$

From state 6 we can now read the $c$. This simply takes us to state 9.
$stSt = \{0, 2, 3, 6, 9\}$
$symSt = \{a, A, b, c\}$

We now find that reading a $d$ from state 9 is a reduce operation, specifically using production 2^[$A \rightarrow Abc$]. The body of this production has length 3, so we remove the top 3 items from $stSt$ and $symSt$.
$stSt = \{0, 2\}$
$symSt = \{a\}$
We now have to add the driver of production 2 to $symSt$ and the state reached by reading it to $stSt$.
$stSt = \{0, 2, 3\}$
$symSt = \{a, A\}$

The next symbol to be read is still $d$ and we're in state 3. The parsing table tells us that this takes us to state 7.
$stSt = \{0, 2, 3, 7\}$
$symSt = \{a, A, d\}$

From state 7, we now have to read $e$. The parsing table tells us that this is a reduction using production 4^[$B \rightarrow d$]. The length of the body of production 4 is 1, so we pop one state from $stSt$ and one symbol from $symSt$:
$stSt = \{0, 2, 3\}$
$symSt = \{a, A\}$
We now have to add the body of production 4 to $symSt$ and the state reached when we read that symbol from state 3^[top of $stSt$] to $stSt$.
$stSt = \{0, 2, 3, 5\}$
$symSt = \{a, A, B\}$

We are still yet to read the $e$, so we'll do that now. We find ourselves in state 5, and the parsing table says we need to move to state 8.
$stSt = \{0, 2, 3, 5, 8\}$
$symSt = \{a, A, B, e\}$

From state 8 we now have to read $\$$, but the parsing table says that this is a reduction using production 1^[$S \rightarrow aABe$]. The length of the body of production 1 is 4, so we pop 4 states off $stSt$ and 4 symbols off $symSt$:
$stSt = \{0\}$
$symSt = \{\}$
We now add the driver of production 1^[$S$] to $symSt$ and read that from the state on top of $stSt$. This takes us from state 0 to state 1:
$stSt = \{0, 1\}$
$symSt = \{S\}$

Finally we can read $\$$, and doing so from state 1 we find the accepting symbol meaning we have finished.

Now, looking back through our working,  we can reconstruct the derivations that were undertaken to reach our input string. We can see that the productions used, in order, were 1, 4, 2, and 3, or rather
$S \implies aABe \implies aAde \implies aAbcde \implies abbcde$
```

#### Conflict resolution
We'll now try to construct the SLR(1) parsing table for the grammar $$E \rightarrow E + E \mid E * E \mid id$$ using the algorithms detailed above.

![[SLR(1) automaton.png]]
We obtain the above automaton, which can be used to construct the following bottom-up parsing table when we number the productions like so:
1. $E \rightarrow E + E$
2. $E \rightarrow E * E$
3. $E \rightarrow id$

|     | id  | +      | *      | $   | E   |
| --- | --- | ------ | ------ | --- | --- |
| 0   | s2  |        |        |     | 1   |
| 1   |     | s3     | s4     | Acc |     |
| 2   |     | r3     | r3     | r3  |     |
| 3   | s2  |        |        |     | 5   |
| 4   | s2  |        |        |     | 6   |
| 5   |     | s3; r1 | s4; r1 | r1  |     |
| 6   |     | s3; r2 | s4; r2 | r2  |     |

Due to the intrinsic amiguity of the grammar, we end up with 4 conflicts in the parsing table.

By looking at the characteristic automata we can easily see that if we're in state 5, we must have the sequence $E + E$ on top of the stack, and the same goes for when we find ourselves in state 6^[although with $E * E$ rather than $E + E$]. In order to get rid of these conflicts, we're going to manually select which move to retain in each case such that the final parsing table conforms to the rules of precedence and associativity for $+$ and $*$.

Through the use of parse trees, we can efficiently ensure that the rules of precedence are respected. Trees are naturally good at such things thanks to their nested nature, and our ability to assume that for a tree to be evaluated all of its child-trees must have already been evaluated.

Specifically, we're interested in implementing the left-associativity of the operators $*$ and $+$, as well as the precedence that $*$ holds over $+$. This means that in the case of $E + E + E$ we want to evaluate $(E + E) + E$ rather than $E + (E + E)$, and in the case of $E * E + E$ we want to evaluate $(E * E) + E$ rather than $E * (E + E)$. In terms of parse trees, the bracketed terms of our expressions should be located "deeper" in the tree than the non-bracketed terms.

Let's finally move on to resolving the conflicts we came across:
- $M[5, +] = s3, r1$
	- This is the situation in which we have $E + E$ on top of our $symSt$^[symbol stack] and we're reading $+$. As $+$ is left-associative, we want to bracket the $E + E$ found on top of $symSt$. Therefore, we choose $r1$ to stay in $M[5, +]$.
- $M[6, *] = s4, r2$
	- We'll choose to look at this cell next as the situation we find ourselves in is analogous to the situation we were in with the previous cell. In this case, we have $E * E$ on top of $symSt$ and we're reading $*$. In order to bracket the term already on the symbol stack, we'll choose to keep the reduce move^[r2] in $M[6, *]$.
- $M[5, *] = s4, r1$
	- Here we find ourselves in what is effectively the opposite situation to the previous two cells we looked at. We've read $E + E$ and we're about to read $*$, so we want to bracket what comes next in the parsing of our grammar. To do this we choose to retain the $s4$ move in the cell $M[5, *]$ so that we can continue reading the string and thus find the entirety of the section we want to bracket.
- $M[6, +] = s4, r2$
	- In this situation we have just read $E * E$ and we're about to read $+$. Therefore, due to the rules of precedence relating to the operators $+$ and $*$, we keep the move $r2$ in $M[6, +]$ in order to bracket what we have just read.

After these changes, our parsing table looks like so:

|     | id  | +   | *   | $   | E   |
| --- | --- | --- | --- | --- | --- |
| 0   | s2  |     |     |     | 1   |
| 1   |     | s3  | s4  | Acc |     |
| 2   |     | r3  | r3  | r3  |     |
| 3   | s2  |     |     |     | 5   |
| 4   | s2  |     |     |     | 6   |
| 5   |     | r1  | s4  | r1  |     |
| 6   |     | r2  | r2  | r2  |     |

#### LR(1)-items
```ad-example
$[A \rightarrow \alpha \cdot \beta, \Delta]$ where $\Delta$ is called the **lookahead set**.
```

The closure of LR(1)-items refines $closure_0(\_)$.
The closure of $\{[A \rightarrow \alpha \cdot B\beta, \Delta]\}$ propagates the symbols following $B$ to those items that are added to the set to close with respect to $B$.
It is easy to see that an LR(1)-item is made up of an LR(0)-item and the lookahead set.

##### Closure of sets of LR(1)-items
This works exactly the same as the [[Parsing#Closure of sets of LR 0 -items|closure of sets of LR(0)-items]], but it also allows us to update the set $\Delta$^[the lookahead set], which will prove itself useful when we have to insert reduction moves in states of LR(1) characteristic automata as it tells us which characters to expect to read next when we're about to make a reduction move.

Calculating the closure is relatively simple, as the first part is just calculating the $closure_0()$ of the first element of the set^[the LR(0)-item]. The second part consists of updating the lookahead set, which gets added to the $closure_0()$ of the LR(0)-item.

```ad-note
Given $P$, a set of LR(1)-items, $closure_1(P)$ represents the smallest possible set of items, with the smallest lookahead set, which satisfies the equation $$closure_1(P) = P \cup \{[B \rightarrow \cdot \gamma, \Gamma]:[A \rightarrow \alpha \cdot B \beta, \Delta] \in closure_1(P) \land B \rightarrow \gamma \in P^{'} \land first(\beta\Delta) \subseteq \Gamma\}$$ where $first(\beta\Delta) = \bigcup_{d \in \Delta}first(\beta d)$, and $P^{'}$ is derived from the set of productions $P$ by adding the production $S^{'} \rightarrow S$.
```

We'll now try to calculate the LR(1) characteristic automa for the following grammar $G$, for which we'll need to calculate $closure_1(P)$.
$S \rightarrow aAd \mid bBd \mid aBe \mid bAe$
$A \rightarrow c$
$B \rightarrow c$

We start from state 0, which will naturally contain $closure_1(\{[S^{'} \rightarrow \cdot S, \{\$\}]\})$. This underlines the fact that once we'll have reached the state containing the accepting item $S^{'} \rightarrow S \cdot$ we'll want to come across the $\$$ which we use as the string terminator.
At the moment the only moment we have in $closure_1(P)$ is $[S^{'} \rightarrow \cdot S, \{\$\}]$, which is of the desired form^[$[A \rightarrow a \cdot B \beta, \Delta]$], meaning we can assume that $\Delta = \{\$\}$ and $\beta = \epsilon$. Therefore, $first(\epsilon \$) = first(\$) = \{\$\} \subseteq \Gamma$^[$\Gamma = first(\beta \Delta)$]. This means that the items we'll get from $closure_1(\{[S^{'} \rightarrow \cdot S, \{\$\}]\})$ are:
$[S^{'} \rightarrow \cdot S, \{\$\}]$
$[S \rightarrow \cdot aAd, \{\$\}]$
$[S \rightarrow \cdot bBd, \{\$\}]$
$[S \rightarrow \cdot aBe, \{\$\}]$
$[S \rightarrow \cdot bAe, \{\$\}]$
```ad-note
These are just the productions with $S$ as their driver.
```
```ad-note
The lookahead set of these productions is only used in the case of a reduction, and has no influence on transitions, meaning that the procedure is largely unaltered compared to that of SLR.
```
Thanks to this, we can confirm the existence of 3 transitions that each lead to a different state, those being $\tau(0, S), \tau(0, a),$ and $\tau(0, b)$.

Starting from the first of these states, state 0, we can see that its kernel is $[S^{'} \rightarrow S \cdot, \{\$\}]$. If we compare this to the pattern we're looking for, we can see that $B$ would be $\epsilon$, so we can do nothing more than note down that the state we'd reach (state 1) will contain the accepting item.

Let's now move onto state 2, $\tau(0, a)$. It's kernel contains the following items:
$[S \rightarrow a \cdot Ad, \{\$\}]$
$[S \rightarrow a \cdot Be, \{\$\}]$
These both satisfy the pattern $[A \rightarrow \alpha \cdot B \beta, \Delta]$. In the first case we have that $A = S, \alpha = a, B = A, \beta = d,$ and $\Delta = \$$ so we can check in our grammar to see if there exist productions of the form $B \rightarrow \gamma$. We find $A \rightarrow c$, so thanks to the formula used to calculate $closure_1(P)$ we know that we have to add an item of the form $[A \rightarrow \cdot c, \Gamma]$ to state 2, where $\Gamma = first(\beta \Delta) = first(d\$) = \{d\}$. Specifically, we add the item $A \rightarrow \cdot c, \{d\}$. Applying the same logic to the second item, we find the production $B \rightarrow c$ in the grammar and we therefore have to add a production of the form $[B \rightarrow \cdot c, \Gamma]$, where $\Gamma = first(\beta \Delta) = first(e\$) = \{e\}$. In the end we add the item $B \rightarrow \cdot c, \{e\}$.
After all of this, state 2 contains the following items:
$[S \rightarrow a \cdot Ad, \{\$\}]$
$[S \rightarrow a \cdot Be, \{\$\}]$
$[A \rightarrow \cdot c, \{d\}]$
$[B \rightarrow \cdot c, \{e\}]$
This means we can add 3 more states, those being $\tau(2, A) = 4, \tau(2, B) = 5,$ and $\tau(2, c) = 6$.

Let's now look at state 3, $\tau(0, b)$. Its kernel contains the following items:
$[S \rightarrow b \cdot Bd, \{\$\}]$
$[S \rightarrow b \cdot Ae, \{\$\}]$
These both follow the pattern defined above, so we're looking for a production of the form $B \rightarrow \gamma$, and we find $A \rightarrow c$ and $B \rightarrow c$. After determining their lookahead sets, we add these to state 3, which now looks like this:
$[S \rightarrow b \cdot Bd, \{\$\}]$
$[S \rightarrow b \cdot Ae, \{\$\}]$
 $[A \rightarrow \cdot c, \{e\}]$
 $[B \rightarrow \cdot c, \{d\}]$
 We can now add the states $\tau(3, B) = 7, \tau(3, A) = 8,$ and $\tau(3, c) = 9$.
 
 Looking at $\tau(2, A) = 4$, the kernel starts off with just the item $[S \rightarrow aA \cdot d, \{\$\}]$. As the $\cdot$ precedes a terminal character, we won't find any productions of the form $[A \rightarrow \alpha \cdot B \beta, \Delta]$ so we do nothing and move on.
 
 The same goes for $\tau(2, B) = 5$, whose kernel starts and ends with the item $[S \rightarrow aB \cdot e, \{\$\}]$.
 
 When we start looking at state 6 ($\tau(2, c)$), things get a bit more interesting. Its kernel is
$[A \rightarrow c \cdot, \{d\}]$
$[B \rightarrow c \cdot, \{e\}]$
In the case of LR(0)-items, we'd now find ourselves with an r/r conflict. In this case however, seeing as we also keep track of the lookahead set, we know that if we find ourselves in state 6 we take a $A \rightarrow c$ reduction if and only if the character we find ourselves reading is $d$. This logic is easily adjusted for $B \rightarrow c$ and $e$.

#### The LRm(1) characteristic automaton
The LRm(1) automaton $AM$ is formed from the LR(1) automaton^[the 'm' stands for merged] $A$. We get the states of this new automata by combining all the items in the states $<P_1, ..., P_n>$ of $A$ with the same LR(0) projections into one single state of $AM$.

Transitions:
- If the state $P$ of $A$ has a $Y$-transition to $Q$ and if $P$ has been merged in $<P_1, ..., P_n>$ and $A$ in $<Q_1, ..., Q_n>$, then there is a $Y$-transition in $AM$ from $<P - 1, ..., P_n>$ to $<Q_1, ...., Q_n>$.

```ad-note
The LRm(1)-automaton has the same number of states and the same layout as the LR(0) automaton.
```

##### Construction of an LALR(1) parsing table
For this we need an LRm(1) characteristic automaton and a lookahead function of the form $$LA(P, A \rightarrow \beta) = \bigcup_{[A \rightarrow \beta \cdot, \Delta_j]}\Delta_j$$
```ad-note
A grammar $G$ is LALR(1) if and only if its LALR(1) parsing table has no conflicts.
```