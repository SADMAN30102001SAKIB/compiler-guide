The lecture itself defines bottom-up parsing as building from the leaves toward the root, i.e. repeatedly **reducing the input back to the start symbol**, and specifically says it constructs a **rightmost derivation in reverse**. 

# Bottom-Up Parsing — shortest full-marks guide

## 0. First: what the past papers are actually asking

| Topic                                                      |                Seen in exams | Priority          |
| ---------------------------------------------------------- | ---------------------------: | ----------------- |
| **SLR(1): LR(0) items → CLOSURE/GOTO → table → prove SLR** | 2011, 2013, 2018, 2019, 2021 | 🔥🔥🔥            |
| **Shift-reduce parsing / perform parsing**                 |       2013, 2016, 2020, 2022 | 🔥🔥🔥            |
| **Handle / handle pruning / identify reduction**           |                   2020, 2024 | 🔥🔥🔥            |
| **Shift-reduce conflict + dangling else**                  |       2011, 2013, 2016, 2024 | 🔥🔥🔥            |
| **Top-down vs bottom-up / why bottom-up**                  |             2017, 2019, 2020 | 🔥🔥              |
| **LR parser ACTION/GOTO / parse using table**              |                   2013, 2021 | 🔥🔥              |
| Full **LALR construction**                                 |                  mainly 2011 | 🟡 old/low return |

The recent trend is especially clear: 2020 asks bottom-up parsing directly, 2021 asks SLR, 2022 asks shift-reduce parsing, 2023 has no direct bottom-up question, and 2024 asks **handle + dangling-else shift/reduce conflict/Yacc**.  

Older papers also contain SLR/LALR table construction, shift-reduce issues, dangling-else, stack parsing and SLR proofs.  

### So spend your time roughly like this:

**40% → SLR construction**
**30% → shift-reduce + handle**
**20% → conflicts/dangling else/Yacc**
**10% → definitions/comparisons/why LR**

---

# 1. WTF is Bottom-Up Parsing?

Suppose input is:

```text
id * id
```

Grammar:

```text
E → E + T | T
T → T * F | F
F → (E) | id
```

Top-down thinks:

> Start with `E`. How can I CREATE `id*id`?

Bottom-up thinks:

> I already have `id*id`. How can I DESTROY/REDUCE it back into `E`?

Mental picture:

```text
id * id
↓
F * id
↓
T * id
↓
T * F
↓
T
↓
E
```

That's literally bottom-up parsing.

**Leaves → root.**

The slides describe exactly this reduction process. 

---

# 2. The ONE sentence you must remember

> **Bottom-up parsing = rightmost derivation in reverse.**

Look at the normal derivation:

```text
E
⇒ T
⇒ T * F
⇒ T * id
⇒ F * id
⇒ id * id
```

Now reverse it:

```text
id * id
⇒ F * id
⇒ T * id
⇒ T * F
⇒ T
⇒ E
```

That's bottom-up.

---

# 3. Shift-Reduce Parsing

Bottom-up parsing is usually implemented using a **stack**.

Only four possible actions matter:

```text
SHIFT
REDUCE
ACCEPT
ERROR
```

### SHIFT

Take next input symbol → put it on stack.

```text
Stack: $
Input: id * id $

SHIFT id

Stack: $ id
Input: * id $
```

### REDUCE

If something on top matches RHS of a production:

```text
F → id
```

then:

```text
$ id
```

becomes

```text
$ F
```

The lecture explicitly defines shifting/reduction this way. 

---

# 4. Full shift-reduce parse — memorize this pattern

For:

```text
E → E+T | T
T → T*F | F
F → (E) | id

Input = id*id
```

| Stack    | Input    | Action         |
| -------- | -------- | -------------- |
| `$`      | `id*id$` | shift `id`     |
| `$ id`   | `*id$`   | reduce `F→id`  |
| `$ F`    | `*id$`   | reduce `T→F`   |
| `$ T`    | `*id$`   | shift `*`      |
| `$ T*`   | `id$`    | shift `id`     |
| `$ T*id` | `$`      | reduce `F→id`  |
| `$ T*F`  | `$`      | reduce `T→T*F` |
| `$ T`    | `$`      | reduce `E→T`   |
| `$ E`    | `$`      | **ACCEPT**     |

The lecture has essentially this exact `id*id` example. 

If they give a completely different grammar, **same algorithm**.

---

# 5. HANDLE — huge exam topic now

This sounds scary but it's stupidly simple.

### Handle = **the thing you should reduce NOW.**

For:

```text
id * id
```

First:

```text
[id] * id
 ↑
handle
```

because:

```text
F → id
```

So reduce it:

```text
F * id
```

Now:

```text
[F] * id
 ↑
handle
```

because:

```text
T → F
```

Continue:

```text
id*id       handle = left id
F*id        handle = F
T*id        handle = right id
T*F         handle = T*F
T           handle = T
E           DONE
```

### Exam definition

> A **handle** is a substring matching the RHS of a production whose reduction represents one step in the reverse of a rightmost derivation.

That's exactly how the slides define it. 

### Important trap

Just because something matches a production does **NOT** mean it is the handle.

Example:

```text
T * id
```

Technically:

```text
E → T
```

so you *could* reduce `T → E`.

But that's WRONG here.

Why?

Because then you get:

```text
E * id
```

and the grammar can't continue correctly.

Correct handle is the second:

```text
id → F
```

giving:

```text
T * F
```

This is exactly why we need a method for recognizing **which possible reduction is correct**.

And that leads to LR parsing.

---

# 6. Handle pruning

Fancy name, simple meaning:

> Keep finding the handle and cutting/reducing it until only the start symbol remains.

```text
id*id
  ↓
F*id
 ↓
T*id
   ↓
T*F
 ↓↓↓
 T
 ↓
 E
```

That whole process = **handle pruning**.

---

# 7. Why do conflicts happen?

Suppose the parser reaches some point and thinks:

> Bro... do I SHIFT more input or REDUCE now?

That's a:

## Shift/Reduce Conflict

```text
SHIFT?
   OR
REDUCE?
```

The slides define it exactly as inability to decide whether to shift or reduce. 

---

## Reduce/Reduce Conflict

Parser says:

> Okay I'll reduce... but WHICH production??

```text
reduce using rule 1?
        OR
reduce using rule 2?
```

Example:

```text
S → A | B
A → a
B → a
```

After reading:

```text
a
```

Should we do:

```text
a → A
```

or

```text
a → B
```

That's reduce/reduce.

---

# 8. Dangling ELSE — VERY IMPORTANT

Grammar:

```text
S → if E then S
  | if E then S else S
  | other
```

Imagine stack has:

```text
if E then S
```

and next input is:

```text
else
```

Two choices.

### Choice 1: REDUCE

```text
if E then S
```

is already a complete short `if`.

### Choice 2: SHIFT

Take the `else`:

```text
if E then S else ...
```

and make the longer rule.

💥 **Shift/Reduce conflict.**

The book explicitly shows this configuration. 

### What should we do?

**SHIFT the `else`.**

Then the `else` belongs to the **nearest unmatched if**.

The book explicitly says resolving the conflict in favor of shifting gives the expected association. 

### 2024 Yacc answer

Exam-safe:

> Yacc normally resolves a shift/reduce conflict by choosing **shift** unless precedence/associativity rules specify otherwise. Therefore, in the dangling-else grammar, `else` is shifted and associated with the nearest unmatched `if`.

That one paragraph can basically answer the conceptual part of the 2024 question.

---

# 9. Why LR parsing exists

Everything until now raises one question:

> How does the machine KNOW whether it should shift or reduce?

LR parser solves it using **states**.

The slides say:

> States are sets of **LR items**. 

---

# 10. LR(0) Item — extremely easy

Production:

```text
A → XYZ
```

Put a dot everywhere:

```text
A → ·XYZ
A → X·YZ
A → XY·Z
A → XYZ·
```

That's it.

Those are LR(0) items. 

### Meaning of the dot

Think of the dot as:

> **How far have I reached?**

```text
A → ·XYZ
```

Nothing seen yet.

```text
A → X·YZ
```

Saw X.

```text
A → XY·Z
```

Saw XY.

```text
A → XYZ·
```

Finished RHS → **might reduce now**.

This mental model makes LR dramatically easier.

---

# 11. Augmented Grammar

Original:

```text
S → ...
```

Add new start:

```text
S' → S
```

Why?

So parser has an unmistakable finish line:

```text
S' → S·
```

means:

# ACCEPT.

The lecture uses exactly this augmented-grammar idea before building the LR(0) automaton. 

---

# 12. CLOSURE — easiest possible explanation

Suppose state contains:

```text
A → α · B β
```

Dot is sitting before nonterminal `B`.

That means:

> "I need a B now."

So add **all productions of B**, with dot at beginning.

Example:

```text
S' → ·E
```

Need an `E`.

Add:

```text
E → ·E+T
E → ·T
```

Now `E→·T` needs `T`.

Add:

```text
T → ·T*F
T → ·F
```

Need `F`.

Add:

```text
F → ·(E)
F → ·id
```

Final state:

```text
I0:

S' → ·E
E  → ·E+T
E  → ·T
T  → ·T*F
T  → ·F
F  → ·(E)
F  → ·id
```

That's **CLOSURE**.

### Memory rule

> **Dot before nonterminal → explode/open that nonterminal's productions.**

The actual CLOSURE rule in the lecture is exactly this. 

---

# 13. GOTO — even easier

Suppose:

```text
A → α · X β
```

and we consume `X`.

Move the dot across X:

```text
A → α X · β
```

Then take **CLOSURE again**.

That's:

```text
GOTO(I, X)
```

### Memory:

**GOTO = move dot + closure.**

The slide defines GOTO as the transition between LR(0) states. 

---

# 14. The whole SLR question in ONE pipeline

If exam says:

> **Show whether grammar is SLR(1)**

or

> **Construct SLR parsing table**

do exactly this:

```text
Grammar
   ↓
1. Augment grammar
   ↓
2. Create LR(0) items
   ↓
3. CLOSURE
   ↓
4. GOTO
   ↓
5. Build all states I0, I1, I2...
   ↓
6. Compute FOLLOW()
   ↓
7. Build ACTION + GOTO table
   ↓
8. Check conflicts
   ↓
No conflict → SLR(1)
Conflict    → NOT SLR(1)
```

This is THE algorithm behind essentially every SLR past-paper variation.

The lecture's SLR algorithm explicitly starts with the augmented grammar and canonical LR(0) collection, then builds ACTION/GOTO. 

---

# 15. How to fill an SLR table

This is the part you should memorize.

Suppose state `Ii` contains:

## Case 1 — dot before TERMINAL

```text
A → α · a β
```

and:

```text
GOTO(Ii,a)=Ij
```

write:

```text
ACTION[i,a] = sj
```

`S` = shift.

---

## Case 2 — completed item

```text
A → α ·
```

Then reduce:

```text
A → α
```

but ONLY under terminals in:

```text
FOLLOW(A)
```

So:

```text
ACTION[i,a] = r(A→α)
for every a ∈ FOLLOW(A)
```

**This is the most important SLR rule.**

---

## Case 3 — start completed

```text
S' → S·
```

write:

```text
ACTION[i,$] = accept
```

---

## Case 4 — transition using NONTERMINAL

If:

```text
GOTO(Ii,A)=Ij
```

write:

```text
GOTO[i,A] = j
```

---

# 16. ACTION vs GOTO

Ultra-simple:

| Part       | Contains        |
| ---------- | --------------- |
| **ACTION** | terminals + `$` |
| **GOTO**   | nonterminals    |

ACTION contains:

```text
s5
r3
acc
error
```

GOTO contains only:

```text
state numbers
```

The lecture table has exactly these two sections. 

---

# 17. How do I know whether grammar is SLR(1)?

After filling the table:

### One box contains:

```text
s4
```

Fine.

Or:

```text
r2
```

Fine.

But one box contains:

```text
s4 / r2
```

💥 Shift/Reduce conflict.

Or:

```text
r2 / r5
```

💥 Reduce/Reduce conflict.

Therefore:

```text
conflict exists
→ NOT SLR(1)
```

No cell has multiple actions:

```text
→ grammar IS SLR(1)
```

That's literally your **proof**.

---

# 18. Tiny complete SLR example

Grammar:

```text
S → CC
C → cC | d
```

Augment:

```text
S' → S
```

Number rules:

```text
(1) S → CC
(2) C → cC
(3) C → d
```

### I₀

```text
S' → ·S
S  → ·CC
C  → ·cC
C  → ·d
```

Transitions:

```text
S → I1
C → I2
c → I3
d → I4
```

### I₁

```text
S' → S·
```

→ accept.

### I₂

```text
S → C·C
C → ·cC
C → ·d
```

### I₃

```text
C → c·C
C → ·cC
C → ·d
```

### I₄

```text
C → d·
```

### I₅

```text
S → CC·
```

### I₆

```text
C → cC·
```

FOLLOW:

```text
FOLLOW(S) = {$}
FOLLOW(C) = {c,d,$}
```

Table:

| State | c  | d  | $   | S | C |
| ----- | -- | -- | --- | - | - |
| 0     | s3 | s4 |     | 1 | 2 |
| 1     |    |    | acc |   |   |
| 2     | s3 | s4 |     |   | 5 |
| 3     | s3 | s4 |     |   | 6 |
| 4     | r3 | r3 | r3  |   |   |
| 5     |    |    | r1  |   |   |
| 6     | r2 | r2 | r2  |   |   |

Any conflict?

**No.**

Therefore:

> **The grammar is SLR(1).**

That same workflow solves the 2018/2019/2021 "show whether SLR(1)" type questions.

---

# 19. Using an LR/SLR table to parse input

Don't overthink it.

Suppose stack top state = `2`

Next input = `*`

Look at:

```text
ACTION[2,*]
```

If:

```text
s7
```

→ shift → push state `7`.

If:

```text
r3
```

→ reduce using production 3.

After reduction:

1. pop corresponding states
2. expose previous state
3. use `GOTO[previous state, LHS]`
4. push new state

If:

```text
acc
```

done.

The lecture gives the same state-stack procedure and LR algorithm. 

---

# 20. Top-down vs Bottom-up — exam version

| Top-down                              | Bottom-up                    |
| ------------------------------------- | ---------------------------- |
| Root → leaves                         | Leaves → root                |
| Start symbol → input                  | Input → start symbol         |
| Expands productions                   | Reduces productions          |
| Leftmost derivation                   | Reverse rightmost derivation |
| LL / predictive                       | LR / shift-reduce            |
| Cannot directly handle left recursion | Can handle left recursion    |
| Smaller grammar class                 | Larger grammar class         |
| Simpler                               | More powerful                |

### "Which is practically implemented?"

For **your compiler-design exam**, answer:

> **Bottom-up LR parsing is widely used because it can handle a larger class of grammars, including left-recursive grammars, requires no backtracking for LR grammars, and detects syntax errors early.**

The slides specifically state that LR handles a proper superset of predictive/LL grammars and is a general non-backtracking shift-reduce method. 

---

# 21. Why bottom-up solves top-down's problems

Top-down problems:

```text
Left recursion
A → Aα | β
```

→ infinite recursion.

Also:

* often requires left factoring
* sometimes cannot decide production easily
* handles smaller grammar class

Bottom-up:

* left recursion is **fine**
* waits until it has seen enough input
* then recognizes a handle
* LR states help choose correct action
* accepts more CFGs

So 2017's question is basically this.

---

# 22. LR(k), SLR, CLR, LALR — DON'T OVERSTUDY

Remember:

```text
LR(k)
```

means:

```text
L = scan Left → right
R = Rightmost derivation in reverse
k = number of lookahead symbols
```

Usually:

```text
LR = LR(1)
```

The lecture explicitly gives these meanings. 

### For this exam, know:

```text
LR
├── SLR
├── CLR
└── LALR
```

Your provided lecture/chapter actually teaches **SLR construction**. It only mentions CLR/LALR as similar; it does **not** teach full LALR table construction. 

So I would **not burn study time on full CLR/LALR construction** right now. The explicit LALR-table question I found is old (2011). Learn the names, but master SLR.

---

# 23. The 5 patterns you need to be able to solve

If you can do these, you're basically covered.

### Pattern A — "Perform bottom-up/shift-reduce parsing"

Do:

```text
Stack | Input | Action
```

Keep:

```text
shift → reduce → ... → accept
```

---

### Pattern B — "Find/identify handle"

Ask:

> Which substring should be reduced **now** to reverse the rightmost derivation?

Then state:

```text
handle = ...
production = A → ...
new sentential form = ...
```

---

### Pattern C — "Is grammar SLR(1)?"

Do:

```text
augment
→ LR(0) items
→ closure/goto
→ FOLLOW
→ ACTION/GOTO
→ check conflicts
```

No conflict = SLR.

---

### Pattern D — "Explain shift/reduce conflict"

Write:

> A shift/reduce conflict occurs when the parser cannot decide whether to shift the next input symbol or reduce the symbols already on the stack.

Then dangling else example.

---

### Pattern E — "Top-down vs bottom-up"

Use the small comparison table above + mention:

```text
top-down = leftmost
bottom-up = reverse rightmost
```

---

# 24. What you genuinely need memorized before the exam

Only these:

```text
BOTTOM-UP = input → start symbol
BOTTOM-UP = rightmost derivation reversed

HANDLE = correct substring to reduce now

4 actions:
SHIFT
REDUCE
ACCEPT
ERROR

S/R conflict = shift OR reduce?
R/R conflict = which reduction?

LR item = production + dot

CLOSURE:
dot before nonterminal → add its productions

GOTO:
move dot over symbol → take closure

SLR:
completed A→α·
→ reduce only under FOLLOW(A)

S'→S·
→ ACCEPT

ACTION = terminals
GOTO = nonterminals

Conflict in SLR table
→ NOT SLR(1)
```

If those lines make sense rather than just being memorized, **you have essentially the entire high-return Bottom-Up Parsing section**.

## Final priority

**Must master:**
`shift-reduce → handle → conflicts → LR(0) item → closure → goto → SLR table/proof → parse using table`

**Know shortly:**
`top-down vs bottom-up → why LR → ACTION/GOTO → LR(k)`

**Don't waste time right now:**
full `CLR/LALR` construction, operator-precedence parser, obscure LR theory not appearing in your current slides/questions.

This is much more concentrated than Top-Down Parsing: the scary-looking **SLR automaton/table is really the only big technical concept**. Everything else is basically one idea: **keep reducing the correct handle until the input becomes the start symbol.**
