This topic is **extremely pattern-based**. The same few operations keep coming back: left recursion, left factoring, FIRST/FOLLOW, predictive table, LL(1), and parsing an input string. In 2019, for example, the paper asks left-recursion elimination and then FIRST/FOLLOW + predictive table + parsing; the 2021–2024 papers repeat essentially the same family of tasks.  

# TOP-DOWN PARSING — EXAM SURVIVAL GUIDE

## 0. What actually matters

| Topic                                 | Priority | What exam usually asks             |
| ------------------------------------- | -------: | ---------------------------------- |
| **FIRST + FOLLOW**                    |   🔥🔥🔥 | Calculate sets                     |
| **Predictive parsing table**          |   🔥🔥🔥 | Construct table                    |
| **LL(1)**                             |   🔥🔥🔥 | Prove/check grammar                |
| **Left recursion removal**            |   🔥🔥🔥 | Immediate + indirect               |
| **Parse input using table**           |   🔥🔥🔥 | Stack/input/action                 |
| **Left factoring**                    |     🔥🔥 | Make grammar suitable for top-down |
| Recursive-descent / predictive parser |     🔥🔥 | Explain/construct                  |
| Problems of top-down parsing          |     🔥🔥 | Theory                             |
| Error recovery                        |       🔥 | Panic/phrase level/etc.            |
| Top-down vs bottom-up                 |       🔥 | Short comparison                   |
| LL(k) details                         |   🟢 Low | Mainly concept                     |

The lectures themselves summarize top-down parsing around **recursive descent → remove left recursion → left factoring → FIRST/FOLLOW → parse table → parsing procedure → LL grammar**. 

---

# 1. WTF is Top-Down Parsing?

Imagine you have:

```text
Grammar:
S → aSb | ε

Input:
aabb
```

You start from **S** and try to grow it until it becomes the input.

```text
S
↓
aSb
↓
aaSbb
↓
aaεbb
↓
aabb ✅
```

That's it.

### Mental picture

```text
        S            ← start/root
      / | \
     ...             ↓
                     ↓ grow downward
   input symbols     ← leaves
```

**Top-down parser = Root → Leaves.**

The lecture explicitly describes it as starting from the CFG's nonterminal and building the parse tree from root to leaves. 

And normally it uses:

> **leftmost derivation**

Meaning: always expand the **leftmost nonterminal first**. 

---

# 2. The 3 Big Problems of Top-Down Parsing

This is a beautiful exam answer.

| Problem                      | Meaning                 | Solution                      |                                   |
| ---------------------------- | ----------------------- | ----------------------------- | --------------------------------- |
| Which nonterminal to expand? | There may be many       | **Leftmost derivation + DFS** |                                   |
| Left recursion               | Parser can loop forever | **Eliminate left recursion**  |                                   |
| Which production to choose?  | `A→...                  | ...` confusion                | **Left factoring + FIRST/FOLLOW** |

These are exactly how your L20/L21 slides organize the topic.  

---

# 3. LEFT RECURSION — MASSIVE EXAM TOPIC

## What is it?

If a variable eventually produces **itself at the left side**:

```text
A → Aα
```

Example:

```text
E → E + T | T
```

Look:

```text
E
→ E + T
→ E + T + T
→ E + T + T + T
→ ...
```

💀 Never consumes input.

That's why **top-down parsers cannot handle left-recursive grammar**. 

---

# 4. Immediate Left Recursion — memorize ONE formula

Given:

```text
A → Aα | β
```

Change to:

```text
A  → βA'
A' → αA' | ε
```

## Memory trick

> **β goes first. α goes into the loop.**

### Example

```text
A → Aa | b
```

Here:

```text
α = a
β = b
```

So:

```text
A  → bA'
A' → aA' | ε
```

Done.

---

## Multiple alternatives

If:

```text
A → Aα1 | Aα2 | β1 | β2
```

then:

```text
A  → β1A' | β2A'
A' → α1A' | α2A' | ε
```

That exact general transformation is in your material. 

### Example

```text
A → Aa | Ab | c | d
```

Answer:

```text
A  → cA' | dA'
A' → aA' | bA' | ε
```

---

# 5. Indirect / Hidden Left Recursion

This looks scary but is just **substitution first → normal left-recursion removal second**.

Your lecture uses exactly this grammar:

```text
S → Aa | b
A → Ac | Sd | ε
```

`S` is indirectly left recursive:

```text
S
→ Aa
→ Sda
```

So S came back at the left. 

## How to solve

We have:

```text
A → Ac | Sd | ε
```

Replace `S` in `Sd` using:

```text
S → Aa | b
```

Therefore:

```text
Sd → Aad | bd
```

So now:

```text
A → Ac | Aad | bd | ε
```

Boom. Now it's just **immediate** left recursion.

Left-recursive:

```text
Ac
Aad
```

Non-left-recursive:

```text
bd
ε
```

Therefore:

```text
S  → Aa | b

A  → bdA' | A'
A' → cA' | adA' | ε
```

### Mental algorithm

```text
INDIRECT recursion
      ↓
substitute other nonterminal
      ↓
IMMEDIATE recursion appears
      ↓
use normal formula
```

The full lecture algorithm does exactly this: order the nonterminals, substitute earlier productions into later ones, then eliminate immediate recursion. 

This is **very important** because essentially this exact hidden-recursion structure appears in the materials/past questions.

---

# 6. LEFT FACTORING

Different problem.

Suppose:

```text
A → abc | abd
```

Input begins:

```text
ab...
```

Parser says:

> Bro 😭 should I select `abc` or `abd`?

Both begin with `ab`.

So pull `ab` outside.

```text
A  → abA'
A' → c | d
```

That's **left factoring**.

---

## Master formula

```text
A → αβ1 | αβ2 | γ
```

becomes:

```text
A  → αA' | γ
A' → β1 | β2
```

Your chapter gives this exact transformation. 

### Memory

> Left recursion = **same variable** at left.

```text
A → Aa
```

> Left factoring = **same prefix**.

```text
A → ab | ac
```

DO NOT mix them up.

---

## Classic `if-else`

Before:

```text
S → if E then S else S
  | if E then S
```

Common prefix:

```text
if E then S
```

Factor:

```text
S  → if E then S S'
S' → else S | ε
```

### Very important trap

**Left factoring does NOT necessarily remove ambiguity.**

Your slide explicitly points this out for the dangling-else grammar. 

---

# 7. The correct order before building a predictive parser

Memorize:

```text
Grammar
  ↓
1. Remove LEFT RECURSION
  ↓
2. LEFT FACTOR
  ↓
3. FIRST
  ↓
4. FOLLOW
  ↓
5. Parsing TABLE
  ↓
6. Check LL(1)
  ↓
7. Parse STRING
```

This pipeline will solve a huge percentage of your semester variations.

---

# 8. FIRST — ridiculously easy once you see it

## Meaning

**FIRST(X) = what terminal can appear FIRST when X generates a string?**

Example:

```text
A → apple | ball
```

Then conceptually:

```text
FIRST(A) = {a, b}
```

Compiler grammar version:

```text
A → aB | cD
```

Therefore:

```text
FIRST(A) = {a, c}
```

---

## Rule 1: Terminal

```text
FIRST(a) = {a}
```

Easy.

---

## Rule 2: ε production

```text
A → ε
```

then:

```text
ε ∈ FIRST(A)
```

FIRST is allowed to contain **ε**. 

---

# 9. The only tricky FIRST rule: nullable symbols

Suppose:

```text
S → ABC

A → a | ε
B → b | ε
C → c
```

Ask:

```text
FIRST(S) = ?
```

Look at A first.

```text
FIRST(A) = {a, ε}
```

Could start with `a`.

But A can disappear!

So look at B:

```text
FIRST(B) = {b, ε}
```

Could start with `b`.

But B can disappear too.

Then C:

```text
FIRST(C) = {c}
```

Therefore:

```text
FIRST(S) = {a, b, c}
```

### Mental rule

Walk from left to right:

```text
A B C D...
↑
take FIRST

Can A become ε?
YES → also inspect B.

Can B become ε?
YES → inspect C.

Stop when one cannot disappear.
```

If **every one** can disappear, then add ε too.

---

# 10. FOLLOW — easiest mental picture

## Meaning

**FOLLOW(A) = what terminal can appear immediately AFTER A?**

Example:

```text
S → A b
```

Obviously:

```text
FOLLOW(A) = {b}
```

That's literally it.

The lecture defines FOLLOW in exactly this way. 

---

# 11. The 3 FOLLOW rules you need

## Rule 1 — Start symbol

If `S` is the start symbol:

```text
$ ∈ FOLLOW(S)
```

`$` = end of input.

---

## Rule 2

If:

```text
A → α B β
```

then:

```text
FIRST(β) - {ε}
```

goes into:

```text
FOLLOW(B)
```

Example:

```text
S → A b
```

so:

```text
b ∈ FOLLOW(A)
```

---

## Rule 3

If:

```text
A → αB
```

B is at the end.

Then:

```text
FOLLOW(A) ⊆ FOLLOW(B)
```

Also same thing if:

```text
A → αBβ
```

but:

```text
β ⇒* ε
```

because everything after B might disappear.

---

# 12. FIRST vs FOLLOW — never confuse again

Think of:

```text
X
```

### FIRST asks:

> What can come **out of X first?**

### FOLLOW asks:

> What can stand **after X?**

Example:

```text
S → A b
A → a | ε
```

Then:

```text
FIRST(A)  = {a, ε}
FOLLOW(A) = {b}
```

---

# 13. Two exam traps

### Trap 1

FOLLOW **never contains ε**.

```text
FIRST → may contain ε
FOLLOW → NEVER ε
```

### Trap 2

`$` is normally introduced through:

```text
FOLLOW(StartSymbol)
```

Not FIRST.

---

# 14. MASTER expression grammar

Your lectures repeatedly use this:

```text
E  → T E'
E' → + T E' | ε

T  → F T'
T' → * F T' | ε

F  → ( E ) | id
```

For it:

| Nonterminal | FIRST       | FOLLOW           |
| ----------- | ----------- | ---------------- |
| E           | `{ (, id }` | `{ ), $ }`       |
| E'          | `{ +, ε }`  | `{ ), $ }`       |
| T           | `{ (, id }` | `{ +, ), $ }`    |
| T'          | `{ *, ε }`  | `{ +, ), $ }`    |
| F           | `{ (, id }` | `{ *, +, ), $ }` |

These exact FIRST/FOLLOW sets appear in the lecture. 

Understand this grammar and half the chapter clicks.

---

# 15. Why do we even need FIRST and FOLLOW?

Suppose:

```text
A → aX | bY
```

Input next symbol = `a`.

Clearly choose:

```text
A → aX
```

**FIRST made that decision.**

But suppose:

```text
A → aX | ε
```

When should we select ε?

That's where **FOLLOW(A)** helps.

So:

> **FIRST = which normal production?**
> **FOLLOW = when is ε safe?**

FIRST and FOLLOW are used to choose the production based on the next input symbol. 

---

# 16. Predictive Parsing Table

Think Excel.

### Rows:

```text
Nonterminals
```

### Columns:

```text
Terminals + $
```

### Cell:

```text
Which production should I use?
```

The lecture defines the table as `M[A,a]`, where A is a nonterminal and `a` is a terminal/lookahead. 

---

# 17. The ONLY table-construction rule you need

For every:

```text
A → α
```

### Case 1

For each:

```text
a ∈ FIRST(α), a ≠ ε
```

put:

```text
M[A,a] = A → α
```

### Case 2

If:

```text
ε ∈ FIRST(α)
```

then for every:

```text
b ∈ FOLLOW(A)
```

put:

```text
M[A,b] = A → α
```

Everything else:

```text
ERROR
```

---

# 18. Build the table for the master grammar

```text
E  → TE'
E' → +TE' | ε
T  → FT'
T' → *FT' | ε
F  → (E) | id
```

Result:

|        | `id`    | `+`       | `*`       | `(`     | `)`    | `$`    |
| ------ | ------- | --------- | --------- | ------- | ------ | ------ |
| **E**  | `E→TE'` | —         | —         | `E→TE'` | —      | —      |
| **E'** | —       | `E'→+TE'` | —         | —       | `E'→ε` | `E'→ε` |
| **T**  | `T→FT'` | —         | —         | `T→FT'` | —      | —      |
| **T'** | —       | `T'→ε`    | `T'→*FT'` | —       | `T'→ε` | `T'→ε` |
| **F**  | `F→id`  | —         | —         | `F→(E)` | —      | —      |

### Beautiful shortcut

For ε production:

```text
E' → ε
```

put ε under:

```text
FOLLOW(E') = {), $}
```

Therefore:

```text
M[E',)] = E'→ε
M[E',$] = E'→ε
```

Same idea for `T'`.

---

# 19. LL(1) — actually stupidly simple

LL(1):

```text
L = scan input Left → Right
L = construct Leftmost derivation
1 = use 1 lookahead symbol
```

That's exactly how your lecture defines it. 

### Human meaning

Looking at **one upcoming token**, the parser must know exactly which rule to choose.

---

# 20. Easiest LL(1) test

After constructing your table:

> **Does any cell contain TWO productions?**

### No

```text
✅ LL(1)
```

### Yes

```text
❌ NOT LL(1)
```

This is probably the easiest exam method.

---

# 21. Formal LL(1) condition

For:

```text
A → α | β
```

must have:

```text
FIRST(α) ∩ FIRST(β) = ∅
```

Meaning the alternatives cannot start with the same possible token.

If:

```text
ε ∈ FIRST(α)
```

then additionally:

```text
FIRST(β) ∩ FOLLOW(A) = ∅
```

And vice versa.

The lecture also notes that **left-recursive or ambiguous grammars cannot be LL(1)** and lead to multiply defined table entries. 

### Exam mental shortcut

```text
LL(1) = NO DOUBLE BOOKING
```

One table cell = maximum one production.

---

# 22. Predictive parser — what is it?

Normal recursive descent may say:

```text
Try rule 1
❌ failed
go backwards
Try rule 2
```

That's **backtracking**.

Predictive parser:

```text
lookahead = a
↓
table/FIRST says use rule 2
↓
directly use rule 2
```

No backtracking.

The slides distinguish recursive descent with backtracking from predictive/recursive descent without backtracking. 

---

# 23. Recursive Descent Parser

One killer sentence:

> **Recursive descent uses one procedure/function for each nonterminal.**

This is exactly the lecture definition. 

For:

```text
E  → TE'
E' → +TE' | ε
```

think:

```text
E() {
    T();
    Eprime();
}
```

And:

```text
Eprime() {
    if lookahead == '+' {
        match('+');
        T();
        Eprime();
    }
}
```

Same idea for T, T′ and F.

### Complete exam-style logic

```text
E():
    T()
    E'()

E'():
    if lookahead == '+':
        match('+')
        T()
        E'()
    else:
        return          // ε

T():
    F()
    T'()

T'():
    if lookahead == '*':
        match('*')
        F()
        T'()
    else:
        return          // ε

F():
    if lookahead == id:
        match(id)

    else if lookahead == '(':
        match('(')
        E()
        match(')')

    else:
        error
```

That's enough to handle a typical **"construct recursive-descent parser"** question.

---

# 24. Non-recursive Predictive Parser

Instead of actual recursive function calls, it uses a:

```text
STACK
```

Components:

```text
Input buffer
Parsing stack
Parsing table M
Output
```

Your slide describes exactly these components. 

Initially:

```text
Input:  string$
Stack:  $S
```

where S = start symbol.

---

# 25. Parsing algorithm — memorize this

Let:

```text
X = stack top
a = current input
```

### Case A: X is terminal

If:

```text
X == a
```

then:

```text
pop X
move input forward
```

### Case B: X is nonterminal

Look:

```text
M[X,a]
```

Suppose:

```text
M[X,a] = X → ABC
```

Then:

```text
pop X
push C, B, A
```

Why backwards?

Because `A` must become stack top first.

### Case C

```text
X = $
a = $
```

→ **ACCEPT ✅**

### Otherwise

→ **ERROR ❌**

---

# 26. Parse `id + id * id`

Using our master grammar.

**Stack top is on the RIGHT here.**

| Stack         | Input       | Action       |
| ------------- | ----------- | ------------ |
| `$ E`         | `id+id*id$` | `E→TE'`      |
| `$ E' T`      | `id+id*id$` | `T→FT'`      |
| `$ E' T' F`   | `id+id*id$` | `F→id`       |
| `$ E' T' id`  | `id+id*id$` | match `id`   |
| `$ E' T'`     | `+id*id$`   | `T'→ε`       |
| `$ E'`        | `+id*id$`   | `E'→+TE'`    |
| `$ E' T +`    | `+id*id$`   | match `+`    |
| `$ E' T`      | `id*id$`    | `T→FT'`      |
| `$ E' T' F`   | `id*id$`    | `F→id`       |
| `$ E' T' id`  | `id*id$`    | match `id`   |
| `$ E' T'`     | `*id$`      | `T'→*FT'`    |
| `$ E' T' F *` | `*id$`      | match `*`    |
| `$ E' T' F`   | `id$`       | `F→id`       |
| `$ E' T' id`  | `id$`       | match `id`   |
| `$ E' T'`     | `$`         | `T'→ε`       |
| `$ E'`        | `$`         | `E'→ε`       |
| `$`           | `$`         | **ACCEPT ✅** |

If you understand this table, you can solve basically any **"show whether predictive parser accepts the input"** variation.

---

# 27. How to answer “importance of FIRST and FOLLOW”

Don't write an essay.

Write:

**FIRST**

* Gives terminals that can begin a string derived from a grammar symbol.
* Helps select the correct production using lookahead.

**FOLLOW**

* Gives terminals that can appear immediately after a nonterminal.
* Helps choose an `ε` production.
* Also helps in predictive-parser error recovery.

Done.

---

# 28. Top-down vs Bottom-up

This appears separately in several papers.

| Top-down                     | Bottom-up                          |
| ---------------------------- | ---------------------------------- |
| Root → leaves                | Leaves → root                      |
| Start from start symbol      | Start from input                   |
| Produces leftmost derivation | Reverse of rightmost derivation    |
| Easier to implement manually | More powerful                      |
| Left recursion problematic   | Can handle left-recursive grammars |
| Predictive/LL parser         | Shift-reduce/LR parser             |

Top-down vs bottom-up is explicitly distinguished in the syntax-analysis material. 

---

# 29. Error Recovery

Past papers ask this enough that you should know the four names.

## 1. Panic-mode recovery

```text
int x = + + + ; int y;
          ERROR
```

Parser throws away tokens until something safe like:

```text
;
}
end
```

Then continues.

> **Skip until synchronizing token.**

Advantages:

* simple
* cannot enter an infinite correction loop easily

Disadvantage:

* may skip useful input/errors

Your material gives `;`, `}`, `end`, etc. as synchronizing-token examples. 

---

## 2. Phrase-level recovery

Make a small local correction:

```text
int x = 5,
```

Maybe parser assumes:

```text
int x = 5;
```

So:

> insert/delete/replace one or a few tokens.

---

## 3. Error productions

Add grammar rules representing common mistakes.

Then when one matches:

```text
recognize mistake
→ report helpful error
→ continue
```

---

## 4. Global correction

Find minimum insertions/deletions/replacements required to turn the whole input into a valid program.

Best theoretical correction.

But:

```text
💀 expensive
```

Your chapter covers error productions and global correction as separate recovery strategies. 

---

# 30. Predictive-parser-specific error detection

Error occurs when:

### 1

Stack top is terminal but doesn't match input.

```text
stack top = '+'
input = '*'
```

or:

### 2

Stack top is nonterminal A but:

```text
M[A,input] = empty/error
```

Your L22/L23 slides give exactly these two cases. 

---

# 31. The MOST IMPORTANT mental connection

Don't memorize these as 7 unrelated topics.

They are ONE STORY:

```text
I want a parser
        ↓
Start from root
        ↓
TOP-DOWN
        ↓
But grammar has problems
        ↓
LEFT RECURSION? → remove it
COMMON PREFIX?  → left factor
        ↓
Now how do I choose a rule?
        ↓
FIRST
        ↓
What if ε is possible?
        ↓
FOLLOW
        ↓
Store all decisions
        ↓
PARSING TABLE
        ↓
Any cell has 2 rules?
     /       \
   YES       NO
    ↓         ↓
not LL(1)    LL(1)
              ↓
       Stack + input
              ↓
            PARSE
```

**That is the whole chapter.**

---

# 32. Exam question → instant reaction

### “Why is left recursion unsuitable for top-down parsing?”

Write:

```text
A → Aα
```

causes the parser to repeatedly expand A without consuming input, causing infinite recursion/loop. Therefore left recursion must be eliminated.

---

### “Eliminate left recursion”

Immediately find:

```text
Aα → α
β → β
```

then:

```text
A  → βA'
A' → αA' | ε
```

---

### “Make grammar suitable for top-down parsing”

Check in this order:

```text
1. Left recursion?
2. Common prefix?
```

Then:

```text
remove recursion
left factor
```

Your notes specifically instruct **left-recursion elimination first, then left factoring, then FIRST**. 

---

### “Find FIRST and FOLLOW”

Think:

```text
FIRST = begins with?
FOLLOW = comes after?
```

---

### “Construct predictive parsing table”

Do:

```text
FIRST production → columns

ε production?
FOLLOW(LHS) → columns
```

---

### “Show LL(1)”

Check:

```text
table collision?
```

No collision:

```text
LL(1) ✅
```

---

### “Parse string”

Write:

```text
Stack | Input | Action
```

and apply the table until `$,$`.

---

### “Predictive parser vs recursive descent?”

```text
Recursive descent:
may require backtracking

Predictive:
recursive descent without backtracking;
uses lookahead/FIRST-FOLLOW
```

---

# 33. Four mistakes that destroy marks

## ❌ Mistake 1

Putting `ε` inside FOLLOW.

Never.

---

## ❌ Mistake 2

For an ε-production, putting it under FIRST.

Example:

```text
A → ε
```

The table entry goes under:

```text
FOLLOW(A)
```

---

## ❌ Mistake 3

Forgetting to reverse RHS when pushing onto stack.

Production:

```text
E → T E'
```

Push:

```text
E' first
T second
```

so T becomes stack top.

---

## ❌ Mistake 4

Thinking left factoring automatically means LL(1).

No.

It only removes common prefixes. The resulting grammar still needs FIRST/FOLLOW/LL(1) checking.

---

# 34. What I'd study if the exam were tomorrow

### Absolutely master

```text
1. Immediate left recursion
2. Indirect left recursion
3. Left factoring
4. FIRST
5. FOLLOW
6. Predictive table
7. LL(1)
8. Stack parsing
```

These are your **money topics**.

### Then learn

```text
9. Recursive descent vs predictive
10. Problems of top-down parsing
11. Error recovery
12. Top-down vs bottom-up
```

### Don't burn time on

* deep LL(k) theory
* long backtracking examples
* memorizing the huge lecture parse-tree animations
* implementation details that don't help table construction

---

# 35. The one-page memory sheet

```text
TOP DOWN
Root → leaves
Leftmost derivation


LEFT RECURSION
A → Aα | β

becomes:

A  → βA'
A' → αA' | ε


LEFT FACTORING
A → αβ1 | αβ2

becomes:

A  → αA'
A' → β1 | β2


FIRST(X)
What can BEGIN X?

May contain ε.


FOLLOW(X)
What can come AFTER X?

Start symbol gets $.
Never contains ε.


TABLE for A → α

FIRST(α)-ε
    → put A→α there

If ε ∈ FIRST(α):
FOLLOW(A)
    → put A→α there


LL(1)
L = Left-to-right
L = Leftmost derivation
1 = one lookahead

Any table cell with >1 rule
    → NOT LL(1)


PARSING
Stack = $S
Input = w$

terminal == input → match
nonterminal → table lookup
$ == $ → ACCEPT


PREDICTIVE PARSER
Recursive descent
+ no backtracking
+ one lookahead


ERROR RECOVERY
Panic
Phrase-level
Error production
Global correction
```

## Bottom line

For **your actual 2011–2024 paper pattern**, I would treat this chapter as roughly:

**~70–75%:**
`left recursion + left factoring + FIRST/FOLLOW + table + LL(1) + parsing`

**~25–30%:**
`recursive descent/predictive concepts + top-down issues + error recovery + top-down vs bottom-up`

The recent papers are especially obvious: 2024 puts left recursion, FIRST/FOLLOW and predictive-table/LL(1) together; 2023 asks predictive parsing plus grammar transformation/FIRST-FOLLOW-table; 2022 again asks predictive parsing and a full FIRST/FOLLOW/table/string-parse problem. 

So **don't study the 180-ish pages as 180 pages**. Conceptually, this whole exam section collapses into that one pipeline:

> **Remove LR → factor → FIRST → FOLLOW → table → LL(1) → parse.**

Master that chain and the wording of the question can change a lot without really changing what you have to do.
