# Semantic Analysis

```ad-abstract
title: Purpose
Compute additional information that is beyond the capabilities of context-free grammars once the syntactic structure is known
```

Typically involves:
- Populating symbol tables after declarations

```ad-example
What is the type of the declared variable?
```
- Performing type inference and type checking on expressions and statements

Two-sided:
- Analysis required to establish correctness
- Analysis to enhance efficiency of the translated program

The best way of describing semantic analysis is to identify attributes/properties that need computing and to write (semantic) rules describing how the computation of attributes is related to the productions of the grammar^[This is called attributed grammar or syntax-directed definition].

Instead of the derivation tree, a better basis for semantic computations is the abstract syntax tree^[a compact representation of the derivation tree] but the generation of the abstract syntax tree can itself be the result of semantic analysis.

## SDDs^[Syntax Directed Definitions]
```ad-abstract
title: Definition
Context-free grammars enriched with attributes and rules to compute them. Used to perform semantic analysis and more.
```

### Attributes
Associated with grammar symbols; can be numbers, types, references to the symbol table, location of a variable in a procedure, object code of a procedure, etc.

Attributes of non-terminals are classified as either [[Semantic analysis#Synthesised attributes|synthesised]] or [[Semantic analysis#Inherited attributes|inherited]].

```ad-note
Attributes of terminals can only be synthesised. They are supplied by the lexical analyser, so there is no rule to compute them.
```

#### Synthesised attributes
Attributes of the driver defined as a function of the attributes of symbols in the production.

#### Inherited attributes
Attributes of non-terminals in the body defined as a function of the attributes of symbols in the production.

### Semantic rules
Associated with productions; typically induce the computation of some attributes as functions of the attributes of other symbols of the production.

### Evaluation order
Define a dependency graph for the SDD:
- Set a node of the dependency graph for each attribute associated with each parse tree node
- For each attribute $X.x$ used to define the attribute $Y.y$, set an edge from the node for $X.x$ to the node for $Y.y$^[The edge means "$X.x$ is needed to define $Y.y$"].

The SDD can be evaluated after a topological sort of the dependency graph.

```ad-warning
If the SDD has both synthesised and inherited attributes there is no guarantee that an evaluation order exists as there may be a cycle in the dependency graph.
```

#### Classes of SDDs for which the existence of a topological sort of the dependency graph is guaranteed
- S-attributed SDDs
	- Only synthesised attributes, can just use postorder visit to evaluate
- L-attributed SDDs
	- For each production $A \rightarrow X_1...X_n$ the definition of $X_j.i$ uses at most
		- Inherited attributes of $A$
		- Inherited or synthesised attributes of the left siblings $X_1...X_{j-1}$

### Evaluation during bottom-up parsing
The main challenge is implementing the translation during parsing ^[vs obtaining the parse tree, annotating it, then evaluating the annotated parse tree]. By far the simplest implementation occurs when the grammar can be parsed by the shift/reduce algorithm and the underlying SDD is S-attributed. Together with the state stack $stSt$ and the symbol stack $symSt$, use a semantic stack $semSt$ to keep records of attributes. Execute the code associated with $A \rightarrow \beta$ when the production is reduced. The needed attributes of the symbols on top of $symSt$ are found at the corresponding positions on top of $semSt$.

SDDs are at the basis of syntax-directed translation schemes, where the computation of attributes goes together with fragments of code that use them.

```ad-example
Given the grammar
$V \rightarrow E$ = $\{print(E.val)\}$
$E \rightarrow E_1 + T = \{E.val = E_1.val + T.val\}$
$E \rightarrow T = \{E.val = T.val\}$
$T \rightarrow T_1 * F = \{T.val = T_1.val * F.val\}$
$T \rightarrow F = \{T.val = F.val\}$
$F \rightarrow (E) = \{F.val = E.val\}$
$F \rightarrow digit = \{F.val = digit.lexval\}$
Run the shift/reduce algorithm on the input "$digit + digit$"
```

### Analysis of a grammar for converting strings to numbers
#### Base 10
Let's start off with the following grammar

| Productions                     | Translation schema                          |
| ------------------------------- | ------------------------------------------- |
| $S \rightarrow Digits$          | $\{print(Digits.v)\}$                       |
| $Digits \rightarrow Digits_1 d$ | $\{Digits.v = Digits_1.v * 10 + d.lexval\}$ |
| $Digits \rightarrow d$          | $\{Digits.v = d.lexval\}$                                            |

This grammar is LALR(1) and its SDD is S-attributed, which we can verify by seeing that the value of the father of each possible subtree is solely dependent on its children and not on its left-brothers^[we can evaluate the tree using a simple post-order traversal].

```ad-question
This is all well and good, but what if we wanted to convert strings into numbers in different bases?
```

#### Naive approach to multiple bases
Let's now try to expand this grammar to allow us to efficiently represent the conversion of strings into numbers of different bases, starting with base 10 (decimal) and base 8 (octal). One possible way of doing this would be the following grammar:
$S \rightarrow Num$
$Num \rightarrow o\ Digits \mid Digits$
$Digits \rightarrow Digits\ d \mid d$
```ad-note
When we find a string starting with $o$ we interpret it as an octal number
```
This grammar is also LALR(1), but we can't synthesise the available information in the same way as before. In fact, if we look at the derivation $S \implies Num \implies o\ Digits$, we can see that the symbol $o$ is in the left-hand subtree while the $Digits$ are in the right-hand subtree, meaning that we can't actually evaluate the tree until we're at the root node. Therefore, this grammar is not an S-attributed SDD.

In order to try and get a better result, we could add two new non-terminals as follows:
$S \rightarrow Num$
$Num \rightarrow O\ Digits \mid D\ Digits$
$O \rightarrow o$
$D \rightarrow \epsilon$
$Digits \rightarrow Digits\ d \mid d$
It doesn't seem like much has changed, but through this we can now use global variables to evaluate the base:

| Productions                      | Translation schema                            |
| -------------------------------- | --------------------------------------------- |
| $S \rightarrow Num$              | $\{print(Num.v)\}$                            |
| $Num \rightarrow O\ Digits$      | $\{Num.v = Digits.v\}$                        |
| $Num \rightarrow D\ Digits$      | $\{Num.v = Digits.v\}$                        |
| $O \rightarrow o$                | $\{base = 8\}$                                |
| $D \rightarrow \epsilon$         | $\{base = 10\}$                               |
| $Digits \rightarrow Digits_1\ d$ | $\{Digits.v = Digits_1.v * base + d.lexval\}$ |
| $Digits \rightarrow d$           | $\{Digits.v = d.lexval\}$                                              |

In this way we can calculate everything without ruining the efficiency.