# Compiler front-end
There are four phases in the front-end of the compiler:
- [[Regular languages and lexical analysis|Lexical analysis]]
- [[Parsing]]
- [[Semantic analysis]]^[static checking: operators applied to compatible operands, return/break statements contained within a while statement, etc]
- Intermediate code generation

Semantic analysis and intermediate code generation can be specified by syntax-directed translations.

Not all translation schemes can be implemented during parsing, but all of them can be implemented by creating an abstract syntax tree and then walking it.

## Intermediate representation
Many possibilities:
- Graph-like structures
	- Abstract syntax trees or directed acyclic graphs
- Three-address code
	- `x = y op z`
- Based on another language
	- Often C, which is favoured for its flexibility

## Intermediate code
We could translate
```c
if (x < 100 || x > 200 && x != y) x = 0;
```
to
```wasm
	if x < 100 goto L2
	goto L3
L3:	if x > 200 goto L4
	goto L1
L4:	if x != y goto L2
	goto L1
L2:	x = 0
L1:
```
```ad-note
This intermediate code implements lazy evaluation
```

## Control flow statements
Attributes for statements:
- $S.next$
	- Inherited
	- Marks the beginning of the code that must be executed after $S$ is finished
- $S.code$
	- Synthesised
	- Sequence of intermediate-code steps that implements the statement $S$ and ends with a jump to $S.next$
- $B.true$
	- Inherited
	- Marks the beginning of the code that must be executed it $B$ is true
- $B.false$
	- Inherited
	- Marks the beginning of the code that must be execudet if $B$ is false
- $B.code$
	- Inherited
	- Sequence of intermediate-code steps that implement the boolean condition $B$ and jumps to $B.true$ if $B$ is true, and to $B.false$ otherwise

```ad-note
Booleans have two distinct roles
- Altering the flow of control^[boolean conditions]
- Computing logical values^[boolean expressions, as right-hand sides of assignments]

What we consider here is their translation in the context of control flow statements. When seen as expressions, they are translated analogously to what is done for arithmetic expressions, with logical operators rather than arithmetic operators. The grammar could have distinct non-terminals to distinguish between these two roles.
```

Let's take the grammar:
$P \rightarrow S$
with
$S.next = newlabel()$
$P.code = S.code \triangleright label(S.next)$
where
-	$newlabel$ generates a new label at each invocation
-	The symbol $\triangleright$ stands for the concatenation of intermediate-code fragments
-	$label(L)$ attaches label $L$ to the next three-address instruction to be generated

$S.next$ is the label of the instruction executed when $S$ is finished. If after performing some statements in $S$ we need to jump to another location then we must set up an explicit $goto$ instruction.

### If-statements
$S \rightarrow if (B) S_1$
$B.true = newlabel()$
$B.false = S.next$
$S_1.next = S.next$
$S.next = B.code \triangleright label(B.true) \triangleright S_1.code$

### While-loops
$S \rightarrow while (B) S_1$
$B.true = newlabel()$
$B.false = S.next$
$S_1.next = newlabel()$
$S.code = label(S_1.next) \triangleright B.code \triangleright label(B.true) \triangleright S_1.code \triangleright gen(goto\ S_1.next)$
```ad-note
We place the label pointed to by $S_1.next$ at the start of the block so that we know where to jump to if $B$ evaluates to true at the end of the block. Otherwise we'd just fall through to the rest of $S.code$ or $S.next$ as we no longer want to execute the loop.
```

### Boolean conditions
$B \rightarrow true = gen(goto\ B.true)$
$B \rightarrow false = gen(goto\ B.false)$

#### `OR` conditions
$B \rightarrow B_1 \mid \mid B_2$
$B_1.true = B.true$
$B_1.false = newlabel()$
$B_2.true = B.true$
$B_2.false = B.false$
$B.code = B_1.code \triangleright label(B_1.false) \triangleright B_2.code$
```ad-note
Sticking to the convention of lazy evaluation, if $B_1$ evaluates to true then we jump straight to the label pointed to by $B.true$. Otherwise, we continue to see if $B_2$ will evaluate to true.
```

#### `AND` conditions
$B \rightarrow B_1\ \&\&\ B_2$
$B_1.true = newlabel()$
$B_1.false = B.false$
$B_2.true = B.true$
$B_2.false = B.false$
$B.code = B_1.code \triangleright label(B_1.true) \triangleright B_2.code$
```ad-note
Again sticking to the convention of lazy evaluation, if $B_1$ evaluates to false then we know we can jump straight to the label pointed to by $B.false$, otherwise we need to continue to evaluate $B_2$ before deciding which block of code to execute.
```