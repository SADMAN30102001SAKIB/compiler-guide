Recent papers heavily reward **SDD/SDT, attributed trees, AST, S/L-attributed**, while type-system/type-checking remains the important supporting concept. For example, 2020 asks SDD vs SDT, attributed-tree evaluation and SDT→AST; 2021 again asks SDD/SDT, annotated parse tree and AST construction; 2022 asks inherited attributes, attributed trees and type checking; 2023 asks S/L-attributed, the four type-system components, attributed-tree evaluation and AST via SDT. 

So below is the **minimum complete Semantic Analysis package**. No textbook-tour bullshit. Every idea exists because it helps you solve a recurring question variation.

# SEMANTIC ANALYSIS — FINAL EXAM GUIDE

## 0. The entire chapter in ONE picture

Compiler has already checked:

```text
x = a + b;
```

Parser says:

> “Yep, this sentence follows the grammar.”

But what if:

```text
int x;
string a;
float b;

x = a + b;
```

Grammar is perfectly valid.

But it **doesn't make sense**.

That's Semantic Analysis.

```text
SOURCE
  ↓
Lexical Analysis       → are the words/tokens valid?
  ↓
Syntax Analysis        → is the sentence grammatically valid?
  ↓
SEMANTIC ANALYSIS      → DOES THE SENTENCE MAKE SENSE?
  ↓
IR / Code Generation
```

Semantic checking includes things like **type, size, scope, duplicate declarations, function arguments and return values**. 

### Mental picture

```text
Syntax = grammar police 👮

Semantic = meaning police 🧠
```

Example:

```text
10 + 20        ✅ syntax ✅ meaning
"hello" + 20   ✅ syntax ? meaning depends on language
x = y          ✅ syntax ❌ if y was never declared
f(1,2,3)       ✅ syntax ❌ if f expects 2 arguments
```

That's enough foundation.

---

# 1. ATTRIBUTE GRAMMAR — the heart of the chapter

Suppose grammar says:

```text
E → E1 + T
```

CFG tells us only:

> an expression can contain another expression + term.

But compiler also wants to know:

```text
What is its value?
What is its type?
What code should I generate?
What AST node should I build?
```

So we attach extra information called **attributes**.

Example:

```text
E.val
E.type
E.code
```

And rules for calculating them:

```text
E → E1 + T
E.val = E1.val + T.val
```

That is the central idea.

> **Attribute Grammar = CFG + attributes + semantic rules.**

Your slide defines it exactly as a CFG augmented with rules specifying computations. 

### Mental picture

Normal parse tree:

```text
       E
      /|\
     E + T
```

Attribute grammar puts information on it:

```text
          E.val = 7
           /   \
   E.val=3      T.val=4
```

The tree now has **knowledge attached to it**.

This becomes an:

# **Attributed / Annotated Parse Tree**

Same parse tree + calculated attributes.

---

# 2. THE MOST IMPORTANT DISTINCTION: Synthesized vs Inherited

This is insanely easy once you see the arrows.

## Synthesized = information travels UP ↑

Production:

```text
A → B C
```

Suppose:

```text
A.val = B.val + C.val
```

Picture:

```text
        A.val=7
          ↑
       ↗     ↖
 B.val=3     C.val=4
```

Children give information to parent.

Therefore:

> **Synthesized = CHILD → PARENT.**

Example:

```text
E → E1 + T

E.val = E1.val + T.val
```

`E.val` is synthesized.

Your slide explicitly says synthesized attributes are determined from children and naturally evaluate bottom-up. 

### Memorize

```text
SYNTHESIZED
     ↑
 information rises
```

Like smoke **rising upward**.

---

## Inherited = information comes DOWN / from LEFT → RIGHT

Production:

```text
A → B C
```

Suppose:

```text
C.inh = B.val
```

Picture:

```text
        A
       / \
      B → C
          ↓
        C.inh
```

`C` receives information rather than creating it from its children.

Therefore:

> **Inherited = child receives information from parent / permitted siblings.**

Your slide's example has `T′.inh = F.val`, showing information being passed into another symbol. 

### Memorize

```text
Synthesized ↑  SEND UP
Inherited   ↓→ RECEIVE
```

That's basically the entire concept.

---

# 3. How to identify Synthesized/Inherited in ANY RULE

Suppose:

```text
A → B C D
```

Look at the **LEFT SIDE OF `=`**.

### Case 1

```text
A.x = B.x + C.x
```

`A` is the parent.

So:

```text
children → parent
```

✅ **Synthesized**

---

### Case 2

```text
C.x = A.x
```

`C` is a RHS child.

It receives information.

✅ **Inherited**

---

### Case 3

```text
D.x = B.x + C.x
```

`D` is RHS child receiving information.

✅ **Inherited**

Do **not** overthink the formula.

Look at:

> **Whose attribute is being calculated?**

```text
Parent attribute calculated → synthesized
Child attribute calculated  → inherited
```

This shortcut solves most identification questions.

---

# 4. S-attributed vs L-attributed

Students confuse this with synthesized/inherited.

They're different levels:

```text
Synthesized / Inherited
        ↓
type of ATTRIBUTE

S-attributed / L-attributed
        ↓
type of whole SDD
```

---

## S-attributed

Definition:

> An SDD is **S-attributed if ALL attributes are synthesized.**



Example:

```text
E → E1 + T
E.val = E1.val + T.val
```

All information:

```text
children ↑ parent
```

Therefore:

```text
S-attributed
```

And because information goes upward:

```text
evaluate bottom-up
```

### Memory

**S-attributed = Synthesized Only**

Both start with **S**.

---

# 5. L-attributed — ridiculously simple rule

Take:

```text
A → X1 X2 X3
```

L-attributed allows synthesized attributes.

It also allows inherited information to move roughly:

```text
A
↓
X1 → X2 → X3
```

Meaning:

> A child can inherit information from the **parent or things to its LEFT**.

But NOT from a sibling sitting to its **right**. 

### Golden test

```text
A → B C
```

This is okay:

```text
C.inh = B.val
```

because:

```text
B → C
left → right ✅
```

But:

```text
B.inh = C.val
```

means:

```text
B ← C
left ← right ❌
```

Therefore **not L-attributed**.

This exact pattern appears in your slide:

```text
A → B C

B.i = f(C.c, A.s)
```

`B` needs information from `C`, which is sitting to B's **right**.

❌ Not L-attributed. 

### Mental picture

**L = Left information is allowed.**

```text
Parent
  ↓
X1 → X2 → X3
```

Don't send inherited information backwards:

```text
X1 ← X2    ❌
```

---

# 6. The relationship you MUST memorize

```text
S-attributed ⊂ L-attributed
```

Meaning:

> Every S-attributed SDD is also L-attributed.

But:

> Every L-attributed SDD is NOT necessarily S-attributed.

Your slide explicitly gives this relationship. 

Why?

S-attributed has only:

```text
↑ synthesized
```

L-attributed allows:

```text
↑ synthesized
+
→ inherited
```

So L is the bigger family.

---

# 7. THE EXACT EXAM CLASSIFICATION METHOD

Exam gives:

```text
P1:
S → M N
S.val = M.val + N.val
```

Ask S or L?

`S.val` = parent attribute.

```text
M,N ↑ S
```

Synthesized only.

✅ S-attributed
and therefore ✅ L-attributed.

---

Now:

```text
P2:
M → P Q

M.val = P.val * Q.val
P.val = Q.val
```

First rule:

```text
M.val = ...
```

✅ synthesized.

Second:

```text
P.val = Q.val
```

`P` gets value from its RIGHT sibling `Q`.

```text
P ← Q
```

❌ violates L-attributed rule.

Therefore:

> P2 is **not L-attributed**.

This is literally the MCQ example in your lecture slide. 

### Your 10-second algorithm

For every rule:

```text
1. Is LHS-of-equation the grammar parent?
      YES → synthesized.

2. Is it a child?
      → inherited.

3. Any inherited child using something to its RIGHT?
      YES → NOT L-attributed.

4. Are ALL attributes synthesized?
      YES → S-attributed.
```

Done.

---

# 8. SDD vs SDT — STOP confusing these

This repeats constantly in the papers. 2019, 2020 and 2021 explicitly ask it.  

## SDD = WHAT should be calculated

**Syntax-Directed Definition**

```text
E → E1 + T
E.val = E1.val + T.val
```

It attaches:

```text
attributes + semantic rules
```

to grammar productions.

It describes the **relationship**.

---

## SDT = DO THIS when production occurs

**Syntax-Directed Translation**

Example:

```text
E → E1 + T
    { E.node = MakeAdd(E1.node,T.node); }
```

Or yacc-style:

```text
E → E + T
    { $$ = MakeAddNode($1,$3); }
```

This is actual **action/code attached to grammar production**.

Your slides describe ad-hoc SDT as snippets/actions associated with productions, executed during parsing; in bottom-up parsing they can execute on reduction. 

### Best mental distinction

```text
SDD = equation / specification
SDT = executable action
```

Or:

```text
SDD: "Result SHOULD equal A+B."

SDT: "When this production happens,
      RUN this code."
```

---

# 9. `$1`, `$2`, `$3`, `$$` — easiest explanation

Suppose:

```text
E → E + T
```

Positions:

```text
      $1 $2 $3
       ↓  ↓  ↓
E  →  E  +  T
```

And:

```text
$$
↓
E on the LEFT
```

So:

```text
$$ = $1 + $3
```

means:

```text
parent result =
left E result + T result
```

The slide uses exactly this Yacc convention. 

Memorize:

```text
$$ = LHS

$1 = first RHS symbol
$2 = second RHS symbol
$3 = third RHS symbol
...
```

This becomes extremely important for AST questions.

---

# 10. ATTRIBUTED PARSE TREE — huge exam pattern

Exam gives grammar + SDD and says:

> Build attributed/annotated parse tree and calculate `val`.

Example:

```text
E → E1 + T
E.val = E1.val + T.val

T → digit
T.val = digit.lexval
```

Input:

```text
3 + 4
```

### Step 1: draw normal parse tree

```text
          E
        / | \
       E  +  T
       |     |
       3     4
```

### Step 2: start at tokens

```text
3.lexval = 3
4.lexval = 4
```

### Step 3: apply rules upward

```text
       E.val = 7
       /       \
 E.val=3       T.val=4
    |              |
    3              4
```

That's an attributed parse tree.

Your slide gives the same pattern for arithmetic evaluation such as `3*5+4`, where the values move upward until `L.val = 19`. 

### Exam rule

**Never magically write the final answer.**

Show values at nodes:

```text
leaf → intermediate nodes → root
```

That earns the marks.

---

# 11. How to create an SDD for arithmetic expressions

This question looks difficult but there's one template.

Grammar:

```text
E → E1 + T
E → E1 - T
E → T

T → T1 * F
T → T1 / F
T → F

F → (E)
F → digit
```

Attach `val`.

```text
E → E1 + T     { E.val = E1.val + T.val }
E → E1 - T     { E.val = E1.val - T.val }
E → T          { E.val = T.val }

T → T1 * F     { T.val = T1.val * F.val }
T → T1 / F     { T.val = T1.val / F.val }
T → F          { T.val = F.val }

F → (E)        { F.val = E.val }
F → digit      { F.val = digit.lexval }
```

That's it.

Notice every result climbs upward:

```text
↑ ↑ ↑
```

Therefore this is:

> **S-attributed.**

The same style appears throughout the supplied semantic-analysis material. 

If examiner changes:

```text
+
```

to:

```text
-
*
/
```

you simply change the operation.

---

# 12. DEPENDENCY GRAPH — don't turn this into rocket science

Rule:

```text
E.val = E1.val + T.val
```

What must exist **before** `E.val`?

```text
E1.val
T.val
```

Therefore dependency arrows:

```text
E1.val ─────┐
            ├──→ E.val
T.val  ─────┘
```

Definition:

> A dependency graph shows **which attribute must be known before another attribute can be calculated**.

Your slides describe every semantic rule as defining dependencies among attributes. 

### Universal method

For:

```text
X = f(Y,Z)
```

draw:

```text
Y ─→ X
Z ─→ X
```

Simple.

---

# 13. Why dependency graph matters: evaluation order

Suppose:

```text
a → b → c
```

Then obviously calculate:

```text
a
then b
then c
```

This is a **topological order**.

If there's a cycle:

```text
A → B
↑   ↓
└───┘
```

Then:

```text
A needs B
B needs A
```

Nobody can start.

That's **circularity**.

Your slides state that cyclic attribute-dependence graphs can make normal evaluation impossible. 

For exam, that's basically all you need.

---

# 14. AST — probably your highest-return concept

AST = **Abstract Syntax Tree**.

Take:

```text
a + b * c
```

A parse tree contains grammar plumbing:

```text
E
├─ E
│  └─ T
│     └─ F
│        └─ a
├─ +
└─ T
   ├─ T
   │  └─ F
   │     └─ b
   ├─ *
   └─ F
      └─ c
```

AST throws away all that useless grammar scaffolding:

```text
        +
       / \
      a   *
         / \
        b   c
```

Beautiful.

Your semantic slide defines AST as a common tree-structured IR and gives exactly the rule that operators become nodes while useless grammar productions are collapsed. 

### Mental picture

```text
Parse Tree = grammar's version

AST = programmer/compiler's meaningful version
```

---

# 15. Parse Tree vs AST

Very recurring.

| Parse Tree                           | AST                               |
| ------------------------------------ | --------------------------------- |
| Shows complete grammar structure     | Shows essential program structure |
| Has nonterminals like E,T,F          | Removes useless E,T,F nodes       |
| Includes punctuation/grammar details | Removes unnecessary syntax        |
| Larger                               | Smaller                           |
| Used mainly for parsing              | Useful for later compiler phases  |

Example:

```text
(a+b)
```

Parse tree includes:

```text
(
E
)
```

AST just:

```text
   +
  / \
 a   b
```

That is why past papers say:

> “A parse tree contains much more information than an AST.”

Correct: parse tree preserves grammatical derivation; AST keeps only the meaningful structure.

---

# 16. THE ONE AST CONSTRUCTION TEMPLATE

This is the pattern to memorize.

```text
E → E1 + T
```

AST action:

```text
E.node = MakeAddNode(E1.node, T.node)
```

Yacc version:

```text
E → E + T
{ $$ = MakeAddNode($1,$3); }
```

Similarly:

```text
E → E - T
{ $$ = MakeSubNode($1,$3); }

T → T * F
{ $$ = MakeMulNode($1,$3); }

T → T / F
{ $$ = MakeDivNode($1,$3); }
```

Useless production:

```text
E → T
{ $$ = $1; }
```

Why?

No need to create:

```text
E
|
T
```

Just reuse T's AST.

Parentheses:

```text
F → ( E )
{ $$ = $2; }
```

Again `(` and `)` don't need AST nodes.

Leaves:

```text
F → number
{ $$ = MakeNumNode(token); }

F → id
{ $$ = MakeIdNode(token); }
```

This is essentially the exact AST-building scheme in your lecture. 

### Memorize ONLY this:

```text
operator → MakeOperatorNode(children)

id/number → MakeLeafNode()

useless grammar production → pass child upward
```

Then you can build almost any expression AST.

---

# 17. AST for statements

Suppose:

```c
x = a + b;
```

AST:

```text
        =
       / \
      x   +
         / \
        a   b
```

---

Conditional:

```c
if(a<b)
    x=x+1;
```

Think:

```text
          IF
         /  \
        <    =
       / \  / \
      a  b x   +
             / \
            x   1
```

---

Loop:

```c
while(i<10)
    i=i+1;
```

Mental structure:

```text
          WHILE
          /   \
         <     =
        / \   / \
       i  10 i   +
               / \
              i   1
```

So if examiner invents a different program, don't memorize trees.

Ask:

> **What operation controls everything?**

That becomes the parent.

---

# 18. SDT → AST question

Exam might say:

> Write SDT to construct AST.

Don't panic.

You're just attaching the previous constructors to grammar rules.

Example:

```text
E → E1 + T
    { E.node = MakeAddNode(E1.node,T.node) }

E → T
    { E.node = T.node }

T → id
    { T.node = MakeIdNode(id) }
```

Then draw the final AST.

So:

```text
SDT
 ↓ executes
MakeNode(...)
 ↓
AST
```

That's the entire connection.

2020, 2021 and 2023 all contain direct AST/SDT variants. 

---

# 19. Type System — only the four things that matter

This appeared directly in 2023.

A type system basically needs four components:

### 1. Base types

```text
int
float
double
char
bool
```

### 2. Ways to construct new types

```text
array
pointer
record/struct
list
...
```

### 3. Rules deciding compatibility/equivalence

Can:

```text
int + double
```

work?

Are two structures considered the same type?

### 4. Rules for inferring expression types

Given:

```text
a + b*c
```

what is its resulting type?

These are the four components from your slide.  

Memorize:

> **Types available → build types → compare types → infer types.**

```text
BASE → BUILD → COMPARE → INFER
```

---

# 20. Type inference

This means:

> Determine the type of something.

Example:

```text
2
```

→ `int`

```text
2.5
```

→ `float/double`

```text
int x;
```

→ x is `int`.

Function:

```c
float f()
```

→ return type `float`.

Expression:

```text
a + b
```

→ inspect types of `a` and `b`, then use operator rules.

Your slides cover constants, variables, procedures, operations and complete expressions this way. 

---

# 21. Type checking — the exam way

Suppose:

```text
a : int
b : double

a + b
```

Compiler does:

```text
type(a) = int
type(b) = double

          ↓
     operator rule
          ↓
result type / conversion / error
```

Think of every operator as having a little table:

```text
Operand1   Operand2   Result
--------------------------------
int        int        int
int        double     double / cast
double     double     double
string     int        error (depending language)
```

That is literally how your slides describe operation-type inference. 

---

# 22. Type-checking tree: solve from BOTTOM UP

Expression:

```text
a - 2 * c
```

Suppose:

```text
a : double
2 : int
c : float
```

Tree:

```text
          -
         / \
   double   *
           / \
         int float
```

Start bottom:

```text
int * float
     ↓
   float
```

Then:

```text
double - float
      ↓
    double
```

Potential conversions are inserted where operand types don't match.

This is why type checking naturally feels like:

```text
leaves → operators → root
```

Exactly like synthesized attributes.

---

# 23. How to write type-checking semantic rules

Generic production:

```text
E → E1 op E2
```

Think:

```text
if compatible(E1.type,E2.type)
       E.type = resultType(E1.type,E2.type)
else
       E.type = error
```

Exam version:

```text
E → E1 + E2
{
   E.type = lookup(+,
                   E1.type,
                   E2.type);
}
```

If conversion required:

```text
if E1.type != requiredType
    insert conversion
```

You don't need to invent complicated theory.

It's always:

```text
GET child types
       ↓
CHECK operator table
       ↓
CAST if allowed
       ↓
RESULT TYPE or ERROR
```

---

# 24. Name vs Structural Equivalence — tiny but slide-supported

Two types can be considered equivalent in two ways.

## Name equivalence

Same declared type/name.

```text
type A = ...
type B = ...
```

Different names:

```text
A ≠ B
```

even if structures look identical.

## Structural equivalence

Ignore names.

Compare actual structure.

```text
A = { int x; float y; }
B = { int x; float y; }
```

Same structure:

```text
A ≡ B
```

Your slide explicitly gives these two approaches. 

Memory:

```text
NAME       → compare labels
STRUCTURAL → compare inside
```

That's enough.

---

# 25. SDT for code generation — understand the pattern, don't memorize a new chapter

Past papers sometimes use SDT and ask it to produce TAC/assembly.

Conceptually nothing new happened.

For:

```text
E → E1 + T
```

instead of calculating:

```text
E.val
```

we calculate:

```text
E.code
```

Example concept:

```text
E → E1 + T
{
   generate code for E1;
   generate code for T;
   emit ADD;
}
```

For:

```text
a - b + c
```

mental AST:

```text
       +
      / \
     -   c
    / \
   a   b
```

Therefore operations naturally happen:

```text
a-b
then
result+c
```

SDT simply hangs those code-emission actions on the appropriate grammar reductions.

**Don't create a second mental model for this.**

```text
Same grammar
Same SDT idea

Different thing being produced:
val / type / AST / TAC / assembly
```

That is the real unifying concept.

---

# 26. This is the BIG CONNECTION that makes the whole chapter click

Look at these:

### Calculate value

```text
E → E1 + T
E.val = E1.val + T.val
```

### Calculate type

```text
E → E1 + T
E.type = typeOfPlus(E1.type,T.type)
```

### Build AST

```text
E → E1 + T
E.node = MakeAdd(E1.node,T.node)
```

### Generate code

```text
E → E1 + T
E.code = E1.code || T.code || ADD
```

THEY ARE ALL THE SAME IDEA.

```text
              E
             / \
            E   T
            ↓   ↓
      information from children
              ↓
     do something for '+'
              ↓
       produce parent's
       value/type/node/code
```

Once this clicks, half the chapter stops being separate topics.

---

# 27. SDD vs Attribute Grammar — are they basically the same?

For your exam-thinking:

### Attribute Grammar

Broad formal concept:

```text
CFG + attributes + semantic rules
```

### SDD

Compiler-design terminology for attaching attributes and semantic rules to grammar productions.

So yes, they overlap heavily.

Don't invent some huge conceptual wall between them.

Then:

```text
SDD
 ↓ make actions executable during parsing
SDT
```

Mental hierarchy:

```text
CFG
 │
 ├── + attributes/rules
 │       ↓
 │   Attribute Grammar / SDD
 │
 └── + executable actions
         ↓
        SDT
```

---

# 28. Dependency vs Synthesized/Inherited vs S/L — one final map

These names sound like four chapters.

They're not.

```text
                    ATTRIBUTES
                    /        \
                   /          \
          Synthesized        Inherited
              ↑                ↓/→
           information       information
           goes up           is received
                   \          /
                    \        /
                  WHO DEPENDS
                   ON WHOM?
                      ↓
               Dependency Graph
                      ↓
               evaluation order
```

Then classify whole SDD:

```text
Only synthesized?
      ↓ YES
 S-attributed

Synthesized + legal inherited
(left-to-right)?
      ↓ YES
 L-attributed
```

That's the complete relationship.

---

# 29. AST vs Attributed Parse Tree — VERY IMPORTANT

Don't mix these.

### Attributed Parse Tree

Still the **full parse tree**.

But values/types/etc. are written on nodes.

```text
        E.val=7
       /   |   \
     E     +    T
```

### AST

Throws away grammar junk.

```text
        +
       / \
      3   4
```

So:

```text
Attributed parse tree
= Parse tree + information

AST
= Compressed meaningful structure
```

Completely different ideas.

---

# 30. AST vs Syntax Tree

In these compiler questions, **syntax tree usually means AST / abstract syntax tree** unless the question explicitly creates another distinction.

Do not confuse:

```text
Parse Tree ≠ AST
```

The major exam contrast is always:

```text
Parse tree = detailed grammar
AST       = essential structure
```

---

# 31. Tiny theory insurance

These are in the current slide but deserve only seconds.

### Problems of Attribute Grammar

Main issue:

> **Non-local information becomes awkward.**

You need many copy rules, more storage, a parse tree, and later may need to search the attributed tree for results. 

### Solution / practical alternative

Use:

```text
central table / symbol table
+
ad-hoc syntax-directed actions
```

Then actions can directly read/write information. 

### Ad-hoc SDT advantages

```text
efficient
flexible
avoids AG limitations
```

Disadvantages:

```text
programmer writes more code
programmer handles details directly
```



Don't spend 30 minutes studying this.

---

# 32. FINAL EXAM DECISION TREE

When you see a Semantic Analysis question, identify which box it belongs to.

```text
QUESTION
   │
   ├─ "value/type attached to parse tree?"
   │        → ATTRIBUTED PARSE TREE
   │
   ├─ "identify attribute?"
   │        → parent receives = SYNTHESIZED
   │        → child receives  = INHERITED
   │
   ├─ "S or L attributed?"
   │        → all synthesized = S
   │        → inherited legal left→right = L
   │
   ├─ "dependency graph?"
   │        → RHS of equation → LHS
   │
   ├─ "SDD vs SDT?"
   │        → rules/specification vs executable actions
   │
   ├─ "construct AST?"
   │        → operators = internal nodes
   │        → ids/numbers = leaves
   │        → delete grammar junk
   │
   ├─ "SDT to AST?"
   │        → MakeAdd/MakeMul/etc.
   │
   └─ "type checking/inference?"
            → child types
                ↓
            operator rule
                ↓
            cast/result/error
```

If this decision tree is clear, the questions stop looking unrelated.

---

# 33. The FIVE patterns I'd practice before the exam

You do **not** need 30 solved questions.

Master these five variations:

### Pattern A — classify attributes

```text
A → B C
A.x = B.x + C.x
C.y = B.x
```

Answer:

```text
A.x synthesized
C.y inherited

legal left→right
⇒ L-attributed

not S-attributed because inherited exists
```

---

### Pattern B — detect illegal L-attributed

```text
A → B C

B.x = C.y
```

`B` receives from its right sibling.

```text
B ← C
```

❌ not L-attributed.

---

### Pattern C — attributed tree

Given:

```text
E → E1 + T
E.val = E1.val + T.val
```

and:

```text
2+3+4
```

calculate upward:

```text
2+3 = 5
5+4 = 9
```

Write each `.val` on the corresponding nodes.

---

### Pattern D — AST

```text
a + b*c
```

Always:

```text
       +
      / \
     a   *
        / \
       b   c
```

Precedence is automatically visible.

---

### Pattern E — type checking

```text
a + b*c
```

with:

```text
a : double
b : int
c : float
```

Bottom-up:

```text
b*c
int * float
    ↓
  float

a + result
double + float
      ↓
    double
```

Insert cast wherever the supplied language/operator table requires it.

---

# 34. THE 15-LINE MEMORY SHEET

If you wake up in the exam hall with amnesia, remember this:

```text
Semantic analysis = does syntactically valid code make sense?

Attribute Grammar = CFG + attributes + semantic rules.

Synthesized = child → parent ↑
Inherited   = parent/allowed sibling → child ↓→

S-attributed = ALL synthesized.
L-attributed = synthesized + inherited flowing legally left→right.
Every S is L; every L is NOT S.

Attributed tree = parse tree + calculated attributes.

Dependency: if X=f(Y,Z), draw Y→X and Z→X.
Cycle = circular dependency = problem.

SDD = attributes + semantic rules / WHAT.
SDT = executable actions / DO.

$$ = LHS, $1,$2,$3 = RHS symbols.

AST = essential structure only; remove grammar junk.
Operators = internal nodes; operands = leaves.

Type checking = child types → operator rule → cast/result/error.
```

If those lines **actually make sense to you instead of being memorized**, you understand almost the entire high-return Semantic Analysis section.

---

# 35. What I would actually spend my time on

Based on the historical shift we already identified and the papers I rechecked:

```text
████████████  ~60%
AST
SDD / SDT
Attributed parse-tree construction/evaluation
SDT → AST
Parse tree vs AST

█████         ~25%
Synthesized vs inherited
S-attributed / L-attributed
Type checking / inference / casting

███           ~15%
Type-system 4 components
Dependency graph
Circularity
Name vs structural equivalence
AG limitations / ad-hoc SDT theory
```

This matches the trend: older papers leaned more heavily on type theory, while the recent papers increasingly ask applied **SDD/SDT + AST + attributed-tree** questions.  

I have intentionally **not expanded the little old out-of-slide legacy topics we previously decided to drop**. This guide is the shortest set I'd study for maximum return while still covering the current-slide concepts that can generate the recurring semester-question variations.
