# Finite state automata
- Used to decide whether a word belongs to the language denoted by a regular expression
- [[Finite state automata#NFA nondeterministic finite state automata|NFA]]
- [[Finite state automata#DFA deterministic finite state automata|DFA]]

## NFA^[nondeterministic finite state automata]
- An NFA is a tuple $(S, A, move_n, s_0, F)$ where
	- $S$ is a set of states
	- $A$ is an alphabet with $\epsilon \notin A$
	- $s_0 \in S$ is the **initial** state
	- $F \subseteq S$ is the set of **final** (or **accepting**) states
	- $move_n : S \times (A \cup \{\epsilon\}) \rightarrow 2^S$ is the transition function

### Graphical representation
- $(S, A, move_n, s_0, F)$ is represented as a directed graph where
	- Nodes represent states
	- The initial state is identified by an incoming arrow
	- Final states are identified by a double circle
	- Edges represent the transition function
	- E.g., suppose $move_n(S_1, a) = \{S_2, S_3\}$^[$\{S_2, S_3\}$ is the set of destinations from $S_1$ through $a$], then
	- ![[Simple NFA move function graph.png]]

### Example of an NFA
![[Simple NFA.png]]
Tabular representation:

|       | $\epsilon$  | $a$            | $b$           |
| ----- | ----------- | -------------- | ----------- |
| $S_0$ | $\emptyset$ | $\{S_0, S_1\}$ | $\{S_0\}$   |
| $S_1$ | $\emptyset$ | $\emptyset$    | $\{S_2\}$   |
| $S_2$ | $\emptyset$ | $\emptyset$    | $\{S_3\}$   |
| $S_3$ | $\{S_3\}$   | $\emptyset$    | $\emptyset$ | 

### Accepted languages
- The NFA $N$ **accepts** or (**recognises**) $w$ iff there exists at least one path spelling $w$ from its initial state to one of its final states
- Recall that
	- $\epsilon \epsilon$ spells $\epsilon$
	- $a \epsilon$ spells $a$
	- $\epsilon a$ spells $a$
- The language accepted/recognised by $N$^[written $L(N)$] is the set of all the strings accepted by $N$

#### Example 1
![[example 1.png]]
- Accepted language: $L((a \mid b)^*abb)$
- Would obviously have the same result even if we removed the final $\epsilon$

#### Example 2
![[example 2.png]]
- Accepted language: $L(aa^* \mid bb^*)$

### Thompson's construction
- An algorithm to construct an NFA $N$ from a regular expression $r$ such that $L(N) = L(r)$
- The construction is based on the inductive definition of regular expressions
	- Base: $r$ is either $\epsilon$ or a symbol of the alphabet
		- Define an NFA to recognise $L(\epsilon)$
		- Define an NFA to recognise $L(a)$
	- Step: $r$ is either $r_1 \mid r_2$, $r_1r_2$, $r_{1}^{*}$, or $(r_1)$
		- Given NFAs $N_1$ and $N_2$ such that $L(N_i) = L(r_i)$ for $i = 1, 2$
			- Define an NFA to recognise $L(r_1 \mid r_2)$
			- Define an NFA to recognise $L(r_1r_2)$
			- Define an NFA to recognise $L(r_{1}^{*})$
			- Define an NFA to recognise $L((r_1))$

#### Main features
- Every step of the construction introduces at most 2 new states
	- The generated NFA has at most $2k$ states at most, where $k$ is the number of symbols and operators in the regular expression
- In every intermediate NFA there is
	- Exactly one final state
	- No edge incoming to the initial state
	- No edge outgoing from the final state

#### Base
- $r = a$
	- ![[a.png]]
- $r = \epsilon$
	- ![[epsilon.png]]

#### Step
- $r = r_1 \mid r_2$
	- ![[alternation.png]]
	- Can go through either one to get to the final state
- $r = r_1r_2$
	- ![[concatenation.png]]
	- Have to go through both to get to the final state
- $r = r_{1}^{*}$
	- ![[kleene star.png]]
	- 0 or more occurences of $r_1$ are required to get to the final state ^[$r_1$ is optional but also infinitely applicable]
- $r = (r_1)$
	- ![[parentheses.png]]
	- Equivalent to $r_1$, must pass through $r_1$

#### Complexity
- Constructs an NFA with $n$ nodes and $m$ edges
- Every step adds at most 2 states and 4 edges
- Every step has constant time
- There are $|r|$ steps
- => Space = $n + m = O(|r|)$ and time = $O(|r|)$

#### Example of application
- Take $r = (a \mid b)^*abb$
- $a = r_1$
	- ![[r1.png]]
- $b = r_2$
	- ![[r2.png]]
- $a \mid b = r_1 \mid r_2 = r_3$
	- Apply Thompson's construction for alternation^[can pass through either $r_1$ or $r_2$] to the automata for $r_1$ and $r_2$ to get the automaton for $r_3$
	- ![[r3.png]]
- $(a \mid b) = (r_3) = r_4$, $(a \mid b)^* = r_{4}^{*} = r_5$
	- Apply Thompson's construction for parentheses and for the Kleene star to the automaton for $r_3$ to get the automaton for $r_5$
	- ![[r5.png]]
- $(a \mid b)^*a = r_5r_1 = r_6$
	- Apply Thompson's construction for construction to the automata for $r_5$ and $r_1$ to get the automaton for $r_6$
	- ![[r6.png]]
- $r = (a \mid b)^*abb$
	- Two more steps, the same as the previous one
	- ![[r.png]]

### Simulation of NFA
- Given a word $w$ and an NFA $N$ decide whether $w \in L(N)$
- Let $w = bbb$
- ![[NFA for 'bbb'.png]]
- Reading $bbb$ from $S_0$ is the same as reading it from any of the states in $\{S_0, S_1, S_3, S_5\}$

#### $\epsilon$-closure
- Let $(S, A, move_n, s_0, F)$ be an NFA, $t$ be a state in $S$, and $T$ be a subset of states
- $\epsilon$-closure$(\{t\})$ is the set of states in $S$ which are reachable from $t$ by zero or more $\epsilon$-transitions
- $\epsilon$-closure$(T) = \bigcup_{t \in T} \epsilon$-closure$(\{t\})$

#### Computation of $\epsilon$-closure
- Data structures
	- Stack
	- Boolean array `alreadyOn` of size $|S|$ to check in constant time if a given state $t$ is on the stack
	- Bidimensional array to record $move_n$. Every entry $(t, x)$ is a linked list containing all the states in $move_n(t, x)$
```python
for i in range(1, |S| + 1):
	alreadyOn[i] = False
def closure(t, stack)
	stack.push(t)
	alreadyOn[t] = True
	for u in move_n(t, epsilon):
		if not alreadyOn[u]:
			closure(u, stack);
```

#### Complexity of $\epsilon$-closure
- Each one of the following operations takes constant time
- How many times is each one repeated?
1. `push t onto stack;`
2. `set alreadyOn[t] bit;`
3. `find next u in move_n(t, epsilon);`
4. `test alreadyOn[u] bit;`

- [1.] and [2.] executed at each invocation of `closure()` (either initial or recursive)
	- Every state goes onto stack at most once (alreadyOn bit initially `false`, then set to `true`, never changed again)
	- Overall, assuming that the NFA has $n$ states and $m$ edges, $O(n)$
- [3.] and [4.] executed at each invocation of `closure()` $\forall u \in move_n(t, \epsilon)$
	- In the worst case every state goes onto the stack, and every state has at least one $\epsilon$-transition
	- Overall, assuming that the NFA has $n$ states and $m$ edges, $O(m)$
- Therefore the overall complexity is $O(n + m)$

#### Algorithm
```python
input	: NFA N = (S, A, move_n, s_0, F), w$
output	: "yes" if w in L(N) else "no"
states = epsilon-closure({s_0})
symbol = nextchar()
while symbol != $ do
	states = epsilon-closure(move_n(t, symbol) for t in states)
	symbol = nextchar()
if (states intersect F) != emptyset:
	return "yes"
else
	return "no"
```

##### Example
- ![[r.png]]
- $w = ababb$

| states                           | symbol | $\cup_t move(t, symbol)$ | $\epsilon$-closure         |
| -------------------------------- | ------ | ------------------------ | -------------------------- |
| $T_0 = \{0, 1, 2, 4, 7\}$        | $a$    | $\{3, 8\}$               | $\{1, 2, 3, 4, 6, 7, 8\}$  |
| $T_1 = \{1, 2, 3, 4, 6, 7, 8\}$  | $b$    | $\{5, 9\}$               | $\{1, 2, 4, 5, 6, 7, 9\}$  |
| $T_2 = \{1, 2, 4, 5, 6, 7, 9\}$  | $a$    | $\{3, 8\}$               | $T_1$                      |
| $T_1$                            | $b$    |                          | $T_2$                      |
| $T_2$                            | $b$    | $\{5, 10\}$              | $\{1, 2, 4, 5, 6, 7, 10\}$ |
| $T_3 = \{1, 2, 4, 5, 6, 7, 10\}$ | $      |                          |                            |

##### Complexity
###### Dominant part:
```python
while symbol != $:
	states = epsilon-closure(move_n(t, symbol) for t in states)  # [L1]
	symbol = nextchar()
```

###### Data structures
- Two stacks for states
	- `currentStack` for the current states ("states" on the right-hand side of the assignment at line [L1])
	- `nextStack` for the next states ("states" on the left-hand side of assignment at line [L1])
- Boolean array "alreadyOn" of size $|S|$ to check in constant time whether a state is on the stack
- Bidimensional array to record $move_n$. Every entry $(t, x)$ is a linked list containing all the states in $move_n(t, x)$

###### Complexity of line [L1]
```python
# populate nextStack
for t in currentStack:
	for u in move_n(t, symbol):
		if not alreadyOn[u]:
			closure(u, nextStack)
	currentStack.pop()

# swap nextStack with currentStack
for s in nextStack:
	nextStack.pop()
	currentStack.push(s)
	alreadyOn[s] = false
```

###### Overall complexity
- Assume that the NFA has $n$ states and $m$ edges
- For each while loop
	- Populating `nextStack` is $O(n + m)$
	- Swapping the stacks is $O(n)$
- One while loop costs $O(n + m)$
- Simulation of $w$ costs $O(|w| (n + m))$
- If the NFA results from Thompson's construction, $(n + m)$ is $O(|r|)$, then simulating $w$ is $O(|w| |r|)$

### Pattern matching based on NFAs
- Simulate the NFA
- Actions are associated with final states
- Find the longest match ^[continue simulating until no further move is possible]
- If the reached set of states has associated actions, execute the "first" action
- Otherwise
	- Look backwards to the sequence of states
	- Pick up the first set of states containing at least one final state
	- Execute the "first" action
	- Update the pointer to the input buffer accordingly

```ad-note
Pattern matching based on DFAs obtained by subset construction is analogous.
```

## DFA^[deterministic finite state automata]
- NFA: $(S, A, move_n, s_0, F)$
	- where $move_n : S \times (A \cup \{\epsilon\}) \rightarrow 2^S$
- DFA: $(S, A, move_d, s_0, F)$
	- where $move_d : S \times A \rightarrow S$
- In any DFA
	- There is no $\epsilon$-transition
	- If $move_d$ is **total**, then from every state there is **exactly one** $a$-transition for every $a \in A$
	- If $move_d$ is **partial**, then from every state there is **at most one** $a$-transition for every $a \in A$
- Take the alphabet $\{a\}$
- An example of a DFA with a total transition function is
	- ![[Simple DFA total transition function.png]]
- An example of a DFA with a partial transition function is
	- ![[Simple DFA partial transition function.png]]

### Simulation of DFAs
#### Accepted languages
- The language recognised by a DFA $D$, denoted by $L(D)$, is the set of words $w$ such that
	- Either there is a path spelling $w=a_1...a_k$ with $k \ge 1$ from the initial state of $D$ to some of its final states
	- Or the initial state is also final and $w = \epsilon$

#### Total transition function
- Answer the question $w \in L(D)?$
- Starting from the inital state, follow the path spelling $w$
	- If the reached state is final, then return "yes"
	- Otherwise, return "no"

#### Partial transition function
- Answer the question $w \in L(D)?$
- Starting from the initial state, begin following the path spelling $w = a_1...a_k$
	- If for some $a_i$ there is no target state, then return "no"
	- If $w$ is over and the reached state is final, then return "yes"
	- Otherwise return "no"

#### Partial vs total transition function
- Let $D$ be a DFA with a partial transition function
- Can a DFA $D^{'}$ with a total transition function such that $L(D^{'}) = L(D)$ be defined?
- Use a **sink**
	- Add a sink to the states of $D$
	- Let the sink be the target of all the undefined transitions
	- For every symbol in the alphabet, add the sink with a self-loop
	- ![[Simple DFA partial transition function.png]]
	- becomes
	- ![[Simple DFA with sink.png]]

## Subset construction
- Given the NFA $N$ construct a DFA $D$ such that $L(D) = L(N)$
- Idea
	- Use $\epsilon$-closure to map subsets of states of the NFA into one single state of the DFA
- Define the initial state of the DFA
- Populate the collection of all its states while defining the transition function

```python
for each state T already collected:
	for symbol a in alphabet:
		T' = plausible target of the a-transition from T
		if T' not already in collection
			add T' to collection
			
input	: NFA N = (S, A, move_n, S_0, F)
output	: DFA D = (R, A, move_d, T_0, E) such that L(D) = L(N)
T_0 = epsilon-closure({S_0})
R = {T_0}
set T_0 as unmarked
while some T in R is unmarked:
	mark T
	for a in A:
		T' = epsilon-closure(move_n(t, a) for t in T)
		if T' != empty-set:
			move_d(T, a) = T'
			if T' not in R then
				add T' to R
				set T' as unmarked
for T in R:
	if (T intersect F) != empty-set:
		set T in E
```
### Complexity
- Dominant section is the while loop
- Suppose the NFA has $n$ states and $m$ edges, and the DFA has $n_d$ states
	- while is repeated $n_d$ times
	- for is repeated $|A|$ times
	- $\epsilon$-closure($\bigcup_{t \in T}move_n(t, a))$^[`epsilon-closure(move_n(t, a) for t in T)`] costs $O(n + m)$
	- Then the subsest construction is $O(n_d \times | A | \times (n + m))$
	- The question is: how big can $n_d$ be?

### Example 1 of application 
- From the NFA
- ![[eg1 application of subset construction NFA.png]]

|                                                 | a                                 | b                                 |
| ----------------------------------------------- | --------------------------------- | --------------------------------- |
| $T_0 = \epsilon$-closure$(\{1\}) = \{1, 2, 4\}$ | $\epsilon$-closure$(\{3\}) = T_1$ | $\epsilon$-closure$(\{5\}) = T_2$ |
| $T_1 = \{3\}$                                   | $\epsilon$-closure$(\{3\}) = T_1$ | ...                               |
| $T_2 = \{5\}$                                   | ...                               | $\epsilon$-closure$(\{5\}) = T_2$ |

- We get the DFA
- ![[eg1 application of subset construction DFA.png]]

### Example 2 of application
- From the NFA
- ![[eg2 application of subset construction NFA.png]]

|                                                    | a                                    | b                                                     |
| -------------------------------------------------- | ------------------------------------ | ----------------------------------------------------- |
| $T_0 = \epsilon$-closure$(\{A\}) = \{A, B, C, E\}$ | $\epsilon$-closure$(\{D, E\}) = T_1$ | $\epsilon$-closure$(\{A, E\}) = \{A, B, C, E\} = T_0$ |
| $T_1 = \{D, E\}$                                   | $\epsilon$-closure$(\{E\}) = T_2$    | $\epsilon$-closure$(\{A, B\}) = \{A, B, C, E\} = T_0$ |
| $T_2 = \{E\}$                                      | $\epsilon$-closure$(\{E\}) = T_2$    | $\epsilon$-closure$(\{A\}) = T_0$                                                      |

- We get the DFA
- ![[eg2 application of subset construction DFA.png]]

## DFA minimisation
- Given the DFA $D$ construct a minimal DFA $D^{'}$ such that $L(D^{'}) = L(D)$

### Intuition
- Partition the states of the DFA into equivalent classes
	- Two states $s$ and $t$ are equivalent iff for every $x$ the simulation of $x$ from $s$ is successful iff the simulation of $x$ from $t$ is successful
	- States $s, t$ equivalent $\iff \forall x, sim(x) \in s$ successful $\iff sim(x) \in t$ successful

### State equivalence
- Let $D = (S, A, move_d, s_0, F)$ be a DFA with a total transition function
- Then $s, t \in S$ are equivalent iff $$move_{d}^{*}(s, x) \in F \iff move_{d}^{*}(t, x) \in F, \forall x \in A^*$$ holds, where the multi-step transition function $move_{d}^{*}$ is defined by induction on the length of strings
	- $move_{d}^{*}(s, \epsilon) = s$
	- $move_{d}^{*}(s, wa) = move_d(move_{d}^{*}(s, w), a)$

### Partition refinement
- Partition the states into blocks (subsets of states)
- Start with the two blocks
	- $B_1 = F$
	- $B_2 = S \setminus F$
	- Why this choice for the initial partition?
		- $s \in B_1$ and $t \in B_2$ are not equivalent because $move_{d}^{*}(s, \epsilon) \in F$ and $move_{d}^{*}(t, \epsilon) \notin F$
- Check whether the blocks contain equivalent states
- If not, refine the blocks by splitting them further
- Repeat the checking-refining steps up to the point that the partition cannot be further refined

#### Splitting blocks
- If all the states in $B_i = \{s_1, ..., s_k\}$ are equivalent, then for every $a \in A$ the target states of the $a$-transition from $s_1, ..., s_k$ should belong to the same block
- The block $B_i$ can be split with respect to $(a, B_j)$ if for some $s, t \in B_i, move_d(s, a) \in B_j$ and $move_d(t, a) \notin B_J$
- Splitting the block $B_i$ with respect to $(a, B_j)$ amounts to replacing $B_i$ with the two blocks
	- $\{s \in B_i \mid move_d(s, a) \in B_j\}$
	- $\{s \in B_i \mid move_d(s, a) \notin B_j\}$

#### Algorithm
```python
B1 = F;
B2 = S \ F;
P = {B1, B2};
while some Bi in P can be split with respect to some (a, Bj):
	update P by removing Bi and adding the two blocks
	{s in Bi | move_d(s, a) in Bj} and {s in Bi | move_d(s, a) not in Bj}
```

#### Example
- Couple of great videos by [Neso Academy](https://www.youtube.com/watch?v=0XaGAkY09Wc), much clearer than the slides

## Training
- ![[Initial DFA.png]]
- Find the minimal equivalent DFA
- First find the equivalent DFA, then minimise

|                      |         | a   | b   |
| -------------------- | ------- | --- | --- |
| $A = \{1\}$          | initial | $B$ | $A$ |
| $B = \{1, 2\}$       |         | $C$ | $D$ |
| $C = \{1, 2, 3\}$    |         | $E$ | $F$ |
| $D = \{1, 3\}$       |         | $G$ | $H$ |
| $E = \{1, 2, 3, 4\}$ | final   | $E$ | $F$ |
| $F = \{1, 3, 4\}$    | final   | $G$ | $H$ |
| $G = \{1, 2, 4\}$    | final   | $C$ | $D$ |
| $H = \{1, 4\}$       | final   | $B$ | $A$ |

- DFA resulting from the [[Finite state automata#Subset construction|subset construction]]
- ![[DFA from subset construction.png]]
- Following the algorithm for minimisation detailed in the [Neso Academy](https://www.youtube.com/watch?v=0XaGAkY09Wc) videos, we find that this is already a minimal DFA

## Number of states of DFAs
### Worst case
#### Lemma
For each $n \in \mathbb{N}^+$ there is an NFA with $(n + 1)$ states whose minimal equivalent DFA has at least $2^n$ states and total transition function

#### Proof
- Take $L = L((a \mid b)*a(a \mid b)^{n - 1})$
- There is an NFA accepting $L$ which has exactly $n + 1$ states
- ![[Worst case.png]]
- Suppose, by contradiction, that a minimal DFA $D$ exists which accepts $L$ and has $k \lt 2^n$ states
- There are exactly $2^n$ distinct words over $\{a, b\}$ whose length is $n$
- Then there are two paths in $D$ which
	- Have length $n$
	- Spell $w_1$ and $w_2$ respectively, with $w_1 \neq w_2$
	- Share at least one node
- Then, for some $x_1, x_2, x$, either $$w_1 = x_1ax\ and\ w_2 = x_2bx$$ or $$w_1 = x_1bx\ and\ w_2 = x_2ax$$
- Without loss of generality, suppose that $w_1 = x_1ax$ and $w_2 = x_2bx$
- Then $w_{1}^{'} = x_1ab^{n - 1} \in L(D)$
- Then the state reached by $w_{1}^{'}$ in $D$ is final
- Contradiction - that state cannot be final because it is also reached by $x_2bb^{n-1} \notin L(D)$