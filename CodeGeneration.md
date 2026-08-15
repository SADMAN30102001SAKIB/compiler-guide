This unit has a very clean pattern: **TAC → Basic Blocks/CFG → DAG optimization → Target/Assembly Code** is the main exam pipeline. Recent papers make this even clearer: 2024 directly asks TAC → basic blocks/flow graph → target code, then later DAG → optimized TAC → effect on register allocation/instruction selection.  Similar TAC/CFG/code-generation combinations also appear in 2022–23. 

I’m treating this as **Intermediate Representation + Code Generation + the local optimization needed with them**. Activation records/storage allocation are a separate Runtime Environment cluster.

# The exam map first

| Priority | Topic                                    | What you must be able to do                           |
| -------- | ---------------------------------------- | ----------------------------------------------------- |
| 🔥🔥🔥   | **Three-Address Code (TAC)**             | Convert expressions, `if`, `if-else`, `while`, arrays |
| 🔥🔥🔥   | **Basic Block + CFG + Loop**             | Find leaders → blocks → edges → loops                 |
| 🔥🔥🔥   | **Target/Assembly Code**                 | TAC → `LD/ST/ADD/SUB/BR/...`                          |
| 🔥🔥🔥   | **Simple Code Generator**                | Registers + next-use + descriptors                    |
| 🔥🔥     | **DAG Optimization**                     | Detect common expressions/dead code → optimized TAC   |
| 🔥🔥     | **IR basics / types / 3 axes**           | Short theory question                                 |
| 🔥🔥     | **Quadruple / Triple / Indirect Triple** | Convert and compare                                   |
| 🔥       | **Peephole Optimization**                | Definition + tiny transformations                     |
| 🔥       | **1/2/3-address + Stack IR**             | Recent-paper variation                                |
| Low      | SSA, Call Graph, Dependence Graph        | Only basic idea unless time remains                   |

Older papers repeatedly ask TAC representations, DAG, basic blocks, flow graphs and code generation too, so these are not just recent trends.   

## Study in exactly this order

1. **IR basic idea → graphical/linear/hybrid → 3 axes**
2. **Three-address code**
3. **Quadruple, Triple, Indirect Triple**
4. **1-address / 2-address / 3-address / Stack code**
5. **Basic blocks → leaders → CFG → loops**
6. **DAG → common subexpression/dead-code optimization**
7. **Code-generation issues + target-machine instructions**
8. **Liveness/next-use + register/address descriptors + simple code generator**
9. **Peephole optimization**

If these nine are clear, almost every variation from these papers becomes a remix of something you already know.

---

# 1. WTF is Intermediate Representation (IR)?

Think of a compiler as a translator who doesn't translate English directly to Japanese.

```text
Source Code
    ↓
   IR        ← compiler's internal language
    ↓
Target / Assembly Code
```

The front end produces IR, the middle end improves/optimizes it, and the back end converts it to native code. 

For:

```c
x = a + b * c;
```

High-level source is:

```text
x = a + b*c
```

One possible IR is:

```text
t1 = b * c
x  = a + t1
```

Then assembly-like target code:

```text
LD   R1, b
LD   R2, c
MUL  R1, R1, R2
LD   R2, a
ADD  R1, R2, R1
ST   x, R1
```

### Why IR?

Because the compiler can work on a clean internal version instead of repeatedly going back to source code. The lecture explicitly says the compiler stores facts about the program in IR and then uses that IR rather than referring back to source text. 

Mental picture:

```text
C ─┐
Java ─┼→  IR  → optimizer → x86
Python? ─┘                  → ARM
```

IR separates the **front end** from the **back end**.

---

# 2. The famous “3 axes of IR”

This is especially exam-worthy because 2023 directly asks IR in **three axes**. 

The slides define them as:

```text
1. Structural organization → What does IR LOOK like?
2. Level of abstraction     → How close to source/machine?
3. Naming discipline        → How are temporary values named?
```



### Axis 1: Structure

Three forms:

| Type      | Mental image          | Example   |
| --------- | --------------------- | --------- |
| Graphical | 🌳 / network          | AST, DAG  |
| Linear    | lines of instructions | TAC, ILOC |
| Hybrid    | both                  | CFG       |

The lecture defines hybrid IR as linear code inside blocks plus a graph connecting those blocks. 

### Axis 2: Abstraction

```text
Near SOURCE                       Near MACHINE
     ←──────────────────────────────────→

AST                  TAC                 Assembly
```

Graphical IR tends to be near-source and easy to generate; linear IR tends to be near-machine and easier to optimize. 

### Axis 3: Naming

For:

```text
a - 2*b
```

Many temporary names:

```text
t1 = b
t2 = 2*t1
t3 = a
t4 = t3-t2
```

Reuse names:

```text
t1 = b
t1 = 2*t1
t3 = a
t1 = t3-t1
```

Naming affects optimization and compiler speed. 

That's literally the entire concept.

---

# 3. AST vs DAG vs CFG — never mix them

This distinction is extremely important.

### AST = expression structure

For:

```text
a + b*c
```

```text
       +
      / \
     a   *
        / \
       b   c
```

AST keeps the essential structure and removes unnecessary grammar nodes from a parse tree. 

### DAG = AST + sharing repeated work

Suppose:

```text
x = a+b
y = a+b
```

AST-style idea:

```text
   +          +
  / \        / \
 a   b      a   b
```

DAG says:

```text
       +
      / \
     a   b
     ↑
   x AND y
```

**Same calculation → same node.**

That is the killer idea.

The notes describe DAG as effectively an AST with a unique node for a value; sharing makes redundant repeated computation explicit. 

### CFG = program path

Completely different.

```text
      B1
     /  \
   B2    B3
     \  /
      B4
```

Here nodes are **basic blocks**, not operators.

A CFG has one node for each basic block and edges representing possible transfers of control. 

Remember:

```text
AST → WHAT expression?
DAG → WHAT can be reused?
CFG → WHERE can execution go?
```

---

# 4. Three-Address Code — THE highest-return topic

Three-address code normally looks like:

```text
result = operand1 op operand2
```

Example:

```c
x = a + b*c;
```

Never write:

```text
x = a + b*c     ❌
```

because it contains two operations.

Break it:

```text
t1 = b * c
x  = a + t1
```

That's TAC.

The slides describe three-address operations as essentially:

```text
i ← j op k
```

with two operands and one result. 

## The golden rule

**One important operation per TAC line.**

For:

```text
x = (a+b)*(c-d)
```

```text
t1 = a + b
t2 = c - d
t3 = t1 * t2
x  = t3
```

You can often directly put the final result into `x`:

```text
t1 = a+b
t2 = c-d
x  = t1*t2
```

Both express the same idea.

---

# 5. TAC for `if`

Source:

```c
if (a > b)
    x = y + z;
```

Think:

> If condition is FALSE, skip body.

```text
if a <= b goto L1
x = y + z
L1:
```

Or equivalently:

```text
if a > b goto L1
goto L2
L1:
x = y + z
L2:
```

Use whichever style keeps your logic obvious.

---

# 6. TAC for `if-else`

Source:

```c
if (a > b)
    c = a+b;
else
    c = a-b;

d = c*2;
```

Translation:

```text
if a <= b goto Lelse
c = a + b
goto Lend

Lelse:
c = a - b

Lend:
d = c * 2
```

Mental picture:

```text
condition
  /   \
TRUE FALSE
 |      |
then   else
   \   /
    join
```

2024 literally starts from an `if-else`, asks TAC, then basic blocks, then target machine code. 

---

# 7. TAC for `while`

```c
while (x < 10) {
    x = x + 1;
}
```

Use:

```text
L1:
if x >= 10 goto L2
x = x + 1
goto L1
L2:
```

Mental picture:

```text
     ┌────────────┐
     ↓            │
 condition        │
   ↓ true         │
  body ───────────┘
   ↓ false
  exit
```

Once you understand this, `for` is just:

```text
initialize
L1:
condition
body
increment
goto L1
```

---

# 8. Arrays in TAC/code generation

These are worth knowing because both older and recent papers use array accesses. The 2019 paper asks assembly for `b=a[i]`, `a[j]=i`, then a conditional jump.  The 2020 paper again asks machine instructions involving arrays. 

For a 1-D array with element size `w` bytes:

```text
x = a[i]
```

First compute byte offset:

```text
t1 = i * w
x  = a[t1]
```

For:

```text
a[i] = x
```

```text
t1 = i * w
a[t1] = x
```

For 2-D **row-major** array `A[i][j]` with `N` columns:

```text
element number = i*N + j
byte offset    = (i*N + j)*w
```

So the exam recipe is:

```text
t1 = i * N
t2 = t1 + j
t3 = t2 * w
x  = A[t3]
```

If the question says real numbers need 8 bytes, use `w=8`. If integers are 4 bytes, use `w=4`.

---

# 9. Quadruple, Triple, Indirect Triple

These are **not three different kinds of logic**.

They're just three ways to **store the same TAC**.

Suppose:

```text
t1 = b * c
t2 = a + t1
x  = t2
```

## Quadruple

Four fields:

```text
(op, arg1, arg2, result)
```

|  # | op  | arg1 | arg2 | result |
| -: | --- | ---- | ---- | ------ |
|  0 | `*` | b    | c    | t1     |
|  1 | `+` | a    | t1   | t2     |
|  2 | `=` | t2   | —    | x      |

The slides explicitly define quadruples as `op, arg1, arg2, result`. 

### Why quads are easy

Results have actual names such as `t1`, `t2`.

So moving instructions around is relatively easy. The book notes that quads have explicit names and are easy to reorder. 

---

## Triple

No result field.

The instruction **number itself** is the result.

|  # | op  | arg1  | arg2  |
| -: | --- | ----- | ----- |
|  0 | `*` | b     | c     |
|  1 | `+` | a     | `(0)` |
|  2 | `=` | `(1)` | x     |

`(0)` means:

> result produced by instruction 0.

The slides explicitly say triple results are indicated by the **position of the operation**. 

### Problem

If you reorder instruction numbers, references like `(0)`, `(1)` become annoying to fix.

Triples save space but are harder to rearrange. 

---

## Indirect Triple

Keep the triples, but add a separate list of pointers:

```text
Execution order:
P0 → triple 0
P1 → triple 1
P2 → triple 2
```

To rearrange code, rearrange the **pointer list**, rather than physically moving triples.

So memorize:

```text
Quadruple       → explicit temp names → easy rearrangement
Triple          → position is result   → smaller, hard rearrangement
Indirect Triple → pointer list         → triple + easier rearrangement
```

The lecture labels indirect triples as an improved version of triples. 

---

# 10. One-, Two-, Three-address and Stack code

Recent papers started varying this. The IR slides explicitly cover one-, two-, and three-address models. 

For:

```text
z = x - 2*y
```

### Three-address

```text
t1 = 2 * y
z  = x - t1
```

### Two-address

General form:

```text
x = x op y
```

Example:

```text
t1 = 2
t2 = y
t2 = t2 * t1
z  = x
z  = z - t2
```

The notes show exactly this destructive-operation idea. 

### Stack machine / one-address

Imagine calculator buttons:

```text
PUSH x
PUSH 2
PUSH y
MUL
SUB
STORE z
```

No explicit operands for `MUL`:

```text
stack top two values → multiply → push result
```

That's why it is called a stack-machine representation.

---

# 11. Basic Block — ridiculously easy once you know the 3 leader rules

A **basic block** is basically:

> a straight-line chunk that executes from top to bottom as one unit.

No entering halfway through. No leaving halfway through.

The code-generation chapter describes basic blocks as maximal sequences where control enters through the first instruction and only branches out at the end. 

## THE 3 LEADER RULES

Memorize:

```text
① First instruction                     → leader
② Target of any goto/jump               → leader
③ Instruction immediately AFTER a jump  → leader
```

That's straight from the provided code-generation algorithm. 

---

# 12. Full basic-block example

Given:

```text
1. i = 0
2. if i >= n goto 6
3. x = x + a[i]
4. i = i + 1
5. goto 2
6. y = x
```

Find leaders.

```text
1 → first                        ✓
2 → target of goto 5             ✓
3 → immediately after branch 2   ✓
6 → target of branch 2           ✓
```

Therefore:

```text
B1:
1. i = 0

B2:
2. if i >= n goto 6

B3:
3. x = x + a[i]
4. i = i + 1
5. goto 2

B4:
6. y = x
```

Done.

Most students make this topic harder than it is.

---

# 13. CFG from those blocks

Now each **block becomes one box**.

Add an edge when execution can go from one block to another.

```text
           ┌─────────┐
           │   B1    │
           └────┬────┘
                ↓
           ┌─────────┐
       ┌───│   B2    │──────┐
       │   └─────────┘      │
 false ↓                    ↓ true
   ┌─────────┐          ┌─────────┐
   │   B3    │          │   B4    │
   └────┬────┘          └─────────┘
        │
        └────────→ B2
```

Two edge rules cover almost everything:

```text
B → C if B explicitly jumps to C

OR

B → next block if execution can simply fall through
```

The slide gives exactly those two cases. 

---

# 14. How to find loops

Don't overthink this in normal exam questions.

Look for an edge going **backward to an earlier block**:

```text
B2 → B3 → B2
```

Therefore:

```text
B2, B3
```

form the loop.

The code-generation notes emphasize that loops naturally arise from `while`/`for` and that identifying them in a flow graph matters for optimization. 

So your exam chain is:

```text
TAC
 ↓
find LEADERS
 ↓
make BASIC BLOCKS
 ↓
blocks become CFG NODES
 ↓
connect JUMP + FALL-THROUGH edges
 ↓
spot BACKWARD edges / cycles
```

That chain alone solves a huge number of questions from 2011–2024.

---

# 15. DAG optimization — the entire idea in one example

Basic block:

```text
a = b + c
d = b + c
e = a * d
```

Without thinking:

```text
b+c calculated twice 😭
```

DAG notices both `b+c` are identical:

```text
        *
       / \
      /   \
     +─────+
    / \
   b   c
  tags: a,d
```

So compute `b+c` only once.

Optimized TAC:

```text
a = b + c
d = a
e = a * d
```

That's **common subexpression elimination**.

The code-generation slides specifically build DAGs for local optimization and list common-subexpression elimination, dead-code elimination, statement reordering, and algebraic simplification as basic-block optimizations. 

---

# 16. How to construct a DAG in exam

Suppose:

```text
a = b + c
d = a - e
f = b + c
```

First make leaves for initial values:

```text
b₀   c₀   e₀
```

For:

```text
a = b+c
```

create:

```text
   +(a)
  /   \
b₀    c₀
```

For:

```text
d = a-e
```

```text
       -(d)
      /   \
   +(a)   e₀
```

Now:

```text
f = b+c
```

DO NOT create another `+`.

The existing `+(b,c)` already computes it.

Attach `f` to that node:

```text
   +(a,f)
   /    \
 b₀      c₀
```

This is why DAG detects common subexpressions.

The slides' construction rule is exactly: make initial-value nodes, then for each statement create an operator node whose children represent the most recent definitions of its operands, and attach the variables for which that node is the current definition. 

---

# 17. Dead code with DAG

Suppose:

```text
a = b+c
x = p*q
d = a+1
```

and `x` is never used afterwards and isn't live when the block exits.

Then:

```text
x = p*q
```

is useless.

Delete it.

Definition:

> Dead code computes a value that is never used.

The slide says DAG dead-code elimination removes nodes whose result is not needed by an ancestor and has no attached live variable. 

---

# 18. Why DAG improves register allocation too

2024 asks this indirectly.

Before:

```text
t1 = b+c
t2 = b+c
t3 = t1*t2
```

You have:

```text
3 operations
more temporaries
more simultaneously useful values
```

After DAG:

```text
t1 = b+c
t3 = t1*t1
```

Now:

```text
fewer instructions
fewer temporary values
less register pressure
```

Therefore register allocation becomes easier, and instruction selection has fewer operations to translate.

That's the conceptual answer they want for the 2024 DAG follow-up. 

---

# 19. Now Code Generation

Mental picture:

```text
             Intermediate Code
                    ↓
             ┌──────────────┐
Symbol Table → Code Generator → Target Program
             └──────────────┘
```

Code generation is the final compiler phase and outputs a semantically equivalent target program. 

The goal isn't just:

> generate *some* assembly.

It should:

```text
preserve source meaning
+ be correct
+ use machine resources efficiently
+ execute efficiently
```



---

# 20. Main issues of Code Generation

This is a common 3/4-mark theory answer.

### Target machine

What machine are we generating for?

RISC? CISC? Stack machine?

The slides note RISC tends to have many registers and three-address instructions, while CISC has fewer registers, more addressing modes and often two-address operations. 

### Instruction selection

Given:

```text
a = a + 1
```

Possibility 1:

```text
ADD a, a, 1
```

Possibility 2 if machine supports it:

```text
INC a
```

Both correct.

Second may be cheaper.

**Instruction selection = choose which machine instructions implement the IR.**

The code generator must select a correct and efficient target sequence according to the IR and ISA. 

### Register allocation + assignment

Very important distinction:

```text
Allocation:
Which VARIABLES deserve registers?

Assignment:
WHICH ACTUAL REGISTER does each get?
```

Example:

```text
Allocation → a, b should be kept in registers

Assignment →
a → R1
b → R3
```

The slides explicitly separate these two problems. 

### Evaluation order

For an expression tree, doing left or right subtree first may require different numbers of registers.

So:

```text
same answer
≠
same efficiency
```

The notes specifically state evaluation order can change register needs and target-code efficiency. 

---

# 21. The tiny assembly language you actually need

The supplied Code Generation chapter uses a deliberately simple register machine. 

### LOAD

```text
LD R1, x
```

means:

```text
R1 = memory[x]
```

### STORE

```text
ST x, R1
```

means:

```text
memory[x] = R1
```

These definitions are explicitly used in your slide model. 

### Arithmetic

```text
ADD R1, R2, R3
```

means:

```text
R1 = R2 + R3
```

Likewise:

```text
SUB R1, R2, R3
MUL R1, R2, R3
```

The slide convention is:

```text
OP destination, source1, source2
```



### Unconditional branch

```text
BR L1
```

means:

```text
goto L1
```

### Conditional branch

Example:

```text
BLTZ R1, L1
```

means:

```text
if R1 < 0 goto L1
```



---

# 22. TAC → assembly: easiest possible pattern

Given:

```text
x = y + z
```

Naive safe translation:

```text
LD  R1, y
LD  R2, z
ADD R1, R1, R2
ST  x, R1
```

For:

```text
x = y - z
```

```text
LD  R1, y
LD  R2, z
SUB R1, R1, R2
ST  x, R1
```

You already know 70% of assembly questions now.

---

# 23. But why “Simple Code Generation Algorithm” exists

Imagine:

```text
t = a-b
u = a-c
```

Naive generation:

```text
LD R1,a
LD R2,b
SUB R1,R1,R2

LD R1,a       ← WHY LOAD a AGAIN?!
LD R2,c
SUB R1,R1,R2
```

If `a` is still in a register, reuse it.

That's the purpose of the algorithm:

> **remember what's already inside the registers and avoid stupid loads/stores.**

The slide summarizes the simple code generator exactly that way: process TAC one instruction at a time, track register contents and avoid unnecessary loads/stores. 

---

# 24. Register Descriptor vs Address Descriptor

This sounds scary but is actually stupidly simple.

Imagine:

```text
R1 currently contains a
R2 currently contains t
```

## Register Descriptor asks:

> “What's inside THIS register?”

```text
R1 → {a}
R2 → {t}
R3 → {}
```

## Address Descriptor asks:

> “Where can I find THIS variable's current value?”

```text
a → {memory, R1}
t → {R2}
b → {memory}
```

So:

```text
Register Descriptor
Register → variables

Address Descriptor
Variable → locations
```

The exact definitions are in the code-generation chapter. 

That's all.

---

# 25. Liveness and Next Use

This is the secret behind deciding:

> “Can I destroy/reuse this register?”

Suppose:

```text
1. t = a+b
2. x = t+c
3. y = x+d
```

At line 2:

```text
t is used HERE
```

After line 2, if `t` is never used again:

```text
t is dead
```

So the register containing `t` can be reused.

## Next use means

> At which future instruction will this current value be needed again?

Example:

```text
1. x = a+b
2. c = d+e
3. y = x+1
```

After line 1:

```text
next use of x = line 3
```

If there is no later use:

```text
next use = none
```

Then its register is delicious free real estate.

---

# 26. The backward next-use algorithm

For:

```text
i: x = y + z
```

scan the basic block **BOTTOM → TOP**.

At each instruction:

```text
First record current info for x,y,z.

Then:
x → dead / no next use
y → live / next use = i
z → live / next use = i
```

This is exactly the algorithm in the lecture. 

Why?

Because going backward:

```text
x is DEFINED here
→ old x dies here

y and z are USED here
→ their earlier values must survive until here
```

That's the mental model. Don't memorize it blindly.

---

# 27. How to solve Simple Code Generator questions

For:

```text
x = y op z
```

think:

```text
Do I already have y in a register?
    yes → use it
    no  → load it

Do I already have z?
    yes → use it
    no  → load it

Where should result x go?
    preferably reuse a register whose old value is dead
```

Then:

```text
OP Rx, Ry, Rz
```

The actual slide algorithm uses `getReg()` to choose `Rx`, `Ry`, and `Rz`, loads an operand only if it isn't already in the selected register, and then emits the operator instruction. 

For exam solving, the easiest way to emulate `getReg()` is:

```text
best register = operand register whose old value has no next use
otherwise     = empty/dead register
otherwise     = spill something that must be sacrificed
```

That last heuristic is the practical interpretation needed to reproduce the slide's worked examples.

---

# 28. Real example from your slide

The slide uses:

```text
t = a - b
u = a - c
v = t + u
a = d
d = v + u
```

and three registers.

It begins:

```text
LD  R1, a
LD  R2, b
SUB R2, R1, R2
```

Now:

```text
R1 contains a
R2 contains t
```

Next:

```text
u = a-c
```

`a` is ALREADY in `R1`.

So don't reload it:

```text
LD  R3, c
SUB R1, R1, R3
```

Now `R1 = u`.

Then:

```text
v = t+u
```

`t` is in `R2`, `u` is in `R1`:

```text
ADD R3, R2, R1
```

And the rest proceeds the same way. The worked slide ultimately stores only the values needed on exit. 

This example is basically your template for **all those scary 4-mark register questions**.

---

# 29. What happens at the END of a basic block?

Suppose:

```text
a is live on exit
d is live on exit

t,u,v are only temporaries
```

At the end:

```text
ST a, Ra
ST d, Rd
```

You do **not** waste stores on dead temporaries.

The provided algorithm explicitly says temporary values need not be stored at block exit, whereas live-on-exit variables whose latest values reside in registers must be stored. 

Huge exam clue:

> When the question explicitly tells you “a, d are live on exit; t, u are temporaries,” it is telling you which values require final `ST`s.

---

# 30. Addressing modes you actually need

You don't need to turn this into a whole new chapter.

### Absolute

```text
LD R1, x
```

`x` = memory address/name.

### Register

```text
LD R1, R2
```

value comes from register.

### Immediate

```text
LD R1, #10
```

`#10` = literal number 10.

### Indexed

```text
LD R1, a(R2)
```

means access memory at:

```text
address(a) + R2
```

This is why indexed addressing is perfect for arrays. 

So `a[i]`, after converting `i` to a byte offset in a register, naturally maps to something like:

```text
LD R1, a(R2)
```

---

# 31. Peephole Optimization

Imagine looking at assembly through a tiny window:

```text
... [ 3 or 4 nearby instructions ] ...
```

You notice something dumb and replace it with something shorter/faster.

That's peephole optimization.

The slide defines the peephole as a small sliding window over instructions and repeatedly replaces a sequence with a shorter/faster equivalent. 

### Example: algebraic simplification

```text
ADD R1, R1, #0
```

does nothing.

Delete it.

### Strength reduction

Expensive:

```text
x = y * 2
```

Possible cheaper machine operation:

```text
x = y << 1
```

if the target supports it.

### Machine idiom

Generic:

```text
x = x + 1
```

Machine might have:

```text
INC x
```

The supplied slides explicitly list algebraic simplification, reduction in strength, and machine idioms. 

Definition to write in exam:

> Peephole optimization examines a small sequence of target instructions and replaces it by an equivalent shorter or faster sequence.

Enough.

---

# 32. The whole chapter as ONE mental movie

This is what I want you to see in your head:

```text
SOURCE
│
│  if (a>b)
│      c=a+b;
│  else
│      c=a-b;
│
▼
THREE-ADDRESS CODE
│
│  if a<=b goto L1
│  c=a+b
│  goto L2
│  L1: c=a-b
│  L2:
│
▼
BASIC BLOCKS
│
│   B1 → B2/B3 → B4
│
▼
CONTROL-FLOW GRAPH
│
├── find loops
│
├── inside each block:
│      build DAG
│      eliminate repeated/dead work
│
▼
OPTIMIZED TAC
│
├── check liveness / next use
│
├── choose registers
│
├── register + address descriptors
│
▼
TARGET CODE
│
│  LD
│  ADD / SUB / MUL
│  ST
│  BR / conditional branch
│
▼
PEEPHOLE
│
▼
FASTER TARGET CODE
```

**That is basically the entire exam unit.**

---

# What changed from old papers → recent papers?

The concepts themselves haven't changed much, but the **way they're combined has**.

Earlier papers commonly asked isolated things such as “compare quadruple/triple/indirect triple,” “construct DAG,” “what is IR,” or “find basic blocks.” Examples appear throughout 2011–2017.  

Recent papers increasingly make you **chain concepts together**. 2024 is the clearest example:

```text
source
→ TAC
→ basic blocks
→ CFG
→ target code
```

and separately:

```text
basic block
→ DAG
→ common subexpression removal
→ optimized TAC
→ effect on registers/instruction selection
```



So don't study these as eight disconnected definitions. Study them as **one pipeline**.

---

# What I would NOT spend much time on

**SSA** appeared in an older paper, but I couldn't find Static Single Assignment as an explicit topic in the provided current IR/code-generation slide sets. By contrast, TAC/CFG/DAG/simple code generation are repeatedly supported by both the slides and recent exam papers. So SSA is low ROI unless you've finished everything above.

Similarly, **Call Graph / Dependence Graph** are present in the broader graphical-IR material, but they are nowhere near as important here as AST/DAG/CFG. The lecture does define call graphs as procedures as nodes and calls as edges, for example.  Know the definition; don't sink study time into them.

---

# Your “am I ready?” test

If I give you:

```c
if (a>b) {
    x = b+c;
    y = b+c;
} else {
    x = b-c;
}
z = x*2;
```

you should now be able to go:

```text
Source
   ↓
TAC
   ↓
Quad / Triple / Indirect Triple
   ↓
Leaders
   ↓
Basic Blocks
   ↓
CFG
   ↓
DAG inside blocks
   ↓
remove duplicate b+c
   ↓
optimized TAC
   ↓
LD / ADD / SUB / MUL / ST / BR
   ↓
register reuse using next-use
```

If you can do that **one pipeline**, then questions that look completely different are mostly just changing the source code on top.

**For exam ROI, I would now drill only three numerical/problem types until they're automatic: (1) TAC → leaders → CFG, (2) basic block → DAG → optimized TAC, and (3) TAC → simple code generator with 2/3 registers.** Those are the highest-value problem-solving skills in these uploaded papers.
