Once the mental picture clicks, **activation tree, activation record, recursion stack, access link, display, heap, garbage collection — all become variations of the same idea**.

The two lecture sets define Runtime Environment around storage, procedure calls, recursion, non-local access, parameters, stack/heap management, and garbage collection.  These are also exactly the concepts that dominate the runtime questions in the later exam papers. For example, 2024 asks stack allocation/activation records, activation trees, access links/displays, call/return sequences, and recursive factorial stack growth. 

---

# RUNTIME ENVIRONMENT — FROM ABSOLUTE ZERO

## 0. First understand WTF this chapter is about

Suppose you write:

```c
int add(int a, int b) {
    int c;
    c = a + b;
    return c;
}

int main() {
    int x;
    x = add(10, 20);
}
```

You already know compiler converts the program into machine instructions.

But while the program is **actually running**, some questions appear:

* Where will `x` live?
* Where will `a`, `b`, `c` live?
* When `main()` calls `add()`, how does computer remember where to return?
* How does `add()` receive `10` and `20`?
* What happens to `c` after `add()` finishes?
* What if `add()` calls another function?
* What if a function calls itself 100 times?
* Where are dynamically created objects stored?
* How does an inner function access a variable of an outer function?

The machinery that handles these things is the:

# **Runtime Environment**

The slides define it as the data structures maintained while the target program executes, handling storage layout/allocation, procedure linkages and parameter passing. 

### Mental picture

Think:

```text
COMPILER
   ↓
creates machine code
   ↓
PROGRAM STARTS RUNNING
   ↓
Runtime Environment manages its memory/life
```

Or:

> **Compiler builds the car.
> Runtime Environment manages everything happening while the car is driving.**

That single sentence is enough to start teaching this chapter.

---

# 1. THE BIG MEMORY PICTURE

Before anything else, students need to understand where data lives.

The slides divide runtime memory into roughly:

```text
LOW ADDRESS
┌───────────────────────┐
│         CODE          │  ← machine instructions
├───────────────────────┤
│        STATIC         │  ← globals, static data
├───────────────────────┤
│         HEAP          │  ← dynamically created objects
│          ↓            │
│                       │
│      FREE MEMORY      │
│                       │
│          ↑            │
├───────────────────────┤
│         STACK         │  ← function calls, locals
└───────────────────────┘
HIGH ADDRESS
```

This organization is directly shown in the runtime slides. 

Memorize it as:

> **C S H S**
> **Code → Static → Heap → Stack**

The two things you will use for almost every exam problem are:

**STACK = function calls**

**HEAP = dynamic objects**

---

# 2. THREE STORAGE ALLOCATION STRATEGIES

You will repeatedly see:

1. Static allocation
2. Stack allocation
3. Heap allocation

The slides explicitly separate these strategies. 

---

## 2.1 Static Allocation

Meaning:

> Memory location is decided **before the program runs**.

Example:

```c
int globalX;
```

Imagine compiler says:

```text
globalX will always be at address 500.
```

Done.

### Mental picture

```text
STATIC = RESERVED SEAT

Seat 25 belongs to x.
Nobody changes that.
```

### Advantages

* simple
* fast access
* address known beforehand

### Problems

Everything must basically be predictable beforehand.

Most importantly:

> **Static allocation cannot properly support recursion.**

The slides explicitly list inability to support recursive procedures as a limitation. 

Why?

Suppose:

```c
fact(4)
fact(3)
fact(2)
fact(1)
```

These are **four simultaneous versions of the same function**.

Each needs its own `n`, temporary values, return information etc.

One permanently fixed memory block cannot represent all those independent live calls.

So we need...

---

# 2.2 Stack Allocation

The stack is the superstar of this chapter.

Rule:

> **Function starts → push its information.
> Function ends → pop its information.**

The lecture explains exactly this behavior. 

Example:

```c
main()
   ↓ calls
A()
   ↓ calls
B()
```

Stack:

```text
          TOP
       ┌───────┐
       │   B   │ ← currently running
       ├───────┤
       │   A   │
       ├───────┤
       │ main  │
       └───────┘
```

Now `B` finishes:

```text
       ┌───────┐
       │   A   │ ← top
       ├───────┤
       │ main  │
       └───────┘
```

`A` finishes:

```text
       ┌───────┐
       │ main  │
       └───────┘
```

This is why stack is perfect for procedure calls.

### Magic rule

## **Last called = first returned**

aka LIFO.

```text
CALL ORDER:   main → A → B
RETURN ORDER: B → A → main
```

---

# 2.3 Heap Allocation

Heap is different.

Use heap when data may live independently of the function that created it.

Example:

```c
Object *p = new Object();
```

The object may still be needed long after the current function finishes.

So:

```text
STACK
short-lived function stuff

HEAP
dynamic objects with arbitrary lifetime
```

The slides describe the heap as storage for dynamically created objects and give the usual `allocate/free`, `malloc/free`, `new/delete` idea. 

### Easiest comparison

| Static              | Stack                   | Heap                 |
| ------------------- | ----------------------- | -------------------- |
| decided early       | automatic during calls  | dynamic              |
| fixed addresses     | push/pop                | allocate/free        |
| globals/static data | locals + function calls | objects/dynamic data |
| recursion ❌         | recursion ✅             | arbitrary lifetime ✅ |
| fastest/simple      | cheap                   | more expensive       |

The slides also directly compare stack/heap and static/dynamic behavior. 

---

# 3. PROCEDURE, CALL AND ACTIVATION

Now the real Runtime Environment chapter starts.

Suppose:

```c
void hello() {
    printf("Hi");
}

int main() {
    hello();
    hello();
}
```

There is only **one function definition**:

```text
hello
```

But it executes twice.

Each execution is called an:

# **Activation**

The slides define each execution of a procedure as an activation. 

So:

```text
hello definition = 1

hello activations = 2
```

Think:

> Function = actor
> Activation = one performance of that actor

---

# 4. RECURSION = MULTIPLE LIVE ACTIVATIONS

Example:

```c
fact(3)
```

with:

```c
int fact(int n) {
    if(n == 1)
        return 1;

    return n * fact(n-1);
}
```

Execution:

```text
fact(3)
   calls fact(2)
       calls fact(1)
```

Before `fact(1)` returns, all three exist simultaneously.

```text
fact(3) alive
fact(2) alive
fact(1) alive
```

This is what the slides mean when they say recursion can cause several activations of the same procedure to be alive simultaneously. 

Each one needs separate information:

```text
fact(3): n = 3
fact(2): n = 2
fact(1): n = 1
```

That is why **each activation gets its own activation record**.

But first...

---

# 5. ACTIVATION TREE

This is one of your highest-yield exam topics.

Definition:

> An **activation tree** shows which procedure activation calls which other activation.

The slides define the root as `main`, every node as one activation, and execution corresponds to depth-first traversal. 

---

## Simple example

```c
main() {
    A();
    B();
}

A() {
    C();
}
```

Activation tree:

```text
       main
      /    \
     A      B
     |
     C
```

Interpretation:

```text
main called A
A called C
main later called B
```

### Golden rules

```text
Root = main

Parent = caller

Child = procedure called by parent

Left/right = execution order
```

---

# 6. ACTIVATION TREE VS CALL STACK

Students confuse these constantly.

Use this mental picture:

## Activation Tree = whole movie

## Control Stack = current scene

Suppose whole execution tree:

```text
          main
         /    \
        A      D
       / \
      B   C
```

At the moment `B` is running:

```text
CALL STACK:

B
A
main
```

Why?

Because current path in tree is:

```text
main → A → B
```

The slides say exactly this: the live stack corresponds to the path from the current activation back to the activation-tree root. 

### Huge exam shortcut

> **To find stack at any moment, find the current node in the activation tree and trace back to root.**

That's it.

---

# 7. CONTROL STACK

Definition:

> The runtime stack used to manage procedure calls and returns is called the **control stack**.

Each live procedure activation has one activation record on it. 

Imagine:

```text
main calls A
A calls B
B calls C
```

Then:

```text
TOP
┌───────────────┐
│ AR of C       │
├───────────────┤
│ AR of B       │
├───────────────┤
│ AR of A       │
├───────────────┤
│ AR of main    │
└───────────────┘
BOTTOM
```

When `C` finishes:

```text
POP C
```

When `B` finishes:

```text
POP B
```

And so on.

---

# 8. ACTIVATION RECORD — THE MOST IMPORTANT OBJECT

Every live function call needs information stored somewhere.

That information package is:

# **Activation Record**

also called:

# **Stack Frame / Frame**

Think:

> **Activation = one function call.
> Activation Record = its personal backpack.**

---

# 9. THE 7 FIELDS OF AN ACTIVATION RECORD

The course repeatedly uses the standard seven-field diagram. 

```text
┌────────────────────┐
│ Actual Parameters  │
├────────────────────┤
│ Returned Value     │
├────────────────────┤
│ Control Link       │
├────────────────────┤
│ Access Link        │
├────────────────────┤
│ Saved Machine      │
│ Status             │
├────────────────────┤
│ Local Data         │
├────────────────────┤
│ Temporaries        │
└────────────────────┘
```

Do not memorize seven random words.

Understand them.

---

## 9.1 Actual Parameters

Example:

```c
sum(10, 20);
```

`10` and `20` are actual parameters.

So activation record needs access to them.

Mental picture:

```text
Actual Params
-------------
a = 10
b = 20
```

---

# 9.2 Returned Value

Function:

```c
return a+b;
```

Result has to reach the caller.

Example:

```text
return value = 30
```

---

# 9.3 Control Link

This one is extremely important.

## Control Link points to the caller's activation record.

The slides define it exactly this way. 

Example:

```text
main calls A
A calls B
```

Then:

```text
B.control_link → A
A.control_link → main
```

Mental trick:

# **CONTROL = WHO CALLED ME?**

---

# 9.4 Access Link

Different purpose.

Access Link helps access **non-local variables in lexically enclosing functions**.

Mental trick:

# **ACCESS = WHERE IS MY OUTER SCOPE?**

We'll go deep into this later.

---

# 9.5 Saved Machine Status

Before jumping into another function, CPU may need to remember:

```text
return address
register values
other machine state
```

Imagine:

```text
You're reading page 50.
Someone calls you.

You put a bookmark at page 50.
```

That bookmark is basically the idea behind the saved return address.

The slides explicitly include the return address and register contents under saved machine status. 

---

# 9.6 Local Data

Variables declared inside the function.

```c
void A() {
    int x;
    float y;
}
```

Activation record:

```text
local data:
x
y
```

---

# 9.7 Temporaries

Compiler-generated intermediate values.

For:

```c
x = a + b * c;
```

compiler may internally do:

```text
t1 = b*c
t2 = a+t1
x = t2
```

`t1`, `t2` may need temporary storage.

That's what this field means.

---

# 10. ONE PERFECT MEMORY TRICK FOR ACTIVATION RECORD

Remember function execution as a conversation:

```text
"What did you give me?"       → Actual parameters
"What answer will I give?"    → Returned value
"Who called me?"              → Control link
"Where is my outer scope?"    → Access link
"Where do I return?"          → Saved machine status
"What variables are mine?"    → Local data
"What scratch work do I need?"→ Temporaries
```

If students understand those seven questions, they won't forget the seven fields.

---

# 11. CONTROL LINK VS ACCESS LINK

This is a classic conceptual trap.

Consider:

```text
A
└── B
    └── C
```

Lexically, `C` is written inside `B`.

Now perhaps some other procedure calls `C` indirectly.

### Control Link

Answers:

> **Who called me during execution?**

Dynamic/runtime relationship.

### Access Link

Answers:

> **Which enclosing procedure owns the non-local variables I am allowed to access?**

Lexical nesting relationship.

So:

```text
CONTROL LINK = caller

ACCESS LINK = enclosing scope
```

Never confuse them.

---

# 12. RECURSIVE FACTORIAL — THE UNIVERSAL EXAM EXAMPLE

This one example teaches:

* activation
* activation tree
* activation record
* control stack
* grow/shrink
* return value
* recursion

Consider:

```c
int fact(int n) {
    if(n == 0)
        return 1;

    return n * fact(n-1);
}

int main() {
    fact(4);
}
```

---

## Step 1 — calls

```text
main
 ↓
fact(4)
 ↓
fact(3)
 ↓
fact(2)
 ↓
fact(1)
 ↓
fact(0)
```

Activation tree:

```text
main
 |
fact(4)
 |
fact(3)
 |
fact(2)
 |
fact(1)
 |
fact(0)
```

Because each function only makes one recursive call, the tree is just a chain.

---

# 13. HOW STACK GROWS

Initial:

```text
┌─────────┐
│ main    │
└─────────┘
```

Call `fact(4)`:

```text
┌─────────┐
│ fact(4) │
├─────────┤
│ main    │
└─────────┘
```

Then:

```text
┌─────────┐
│ fact(3) │
├─────────┤
│ fact(4) │
├─────────┤
│ main    │
└─────────┘
```

Eventually:

```text
TOP
┌─────────┐
│ fact(0) │
├─────────┤
│ fact(1) │
├─────────┤
│ fact(2) │
├─────────┤
│ fact(3) │
├─────────┤
│ fact(4) │
├─────────┤
│ main    │
└─────────┘
```

This is **stack growth**.

---

# 14. HOW STACK SHRINKS

`fact(0)`:

```text
returns 1
```

Pop it.

Now:

```text
fact(1) returns 1×1 = 1
```

Pop.

Then:

```text
fact(2) returns 2×1 = 2
```

Then:

```text
fact(3) returns 3×2 = 6
```

Then:

```text
fact(4) returns 4×6 = 24
```

Stack eventually returns to:

```text
┌──────┐
│ main │
└──────┘
```

### Memorize:

```text
CALL  = PUSH
RETURN = POP
```

This exact pattern appears repeatedly in the exam set; for example 2019 asks the complete activation tree and how activation records grow/shrink for recursive factorial. 

---

# 15. HOW TO WRITE ACTIVATION RECORD VALUES IN RECURSION

Suppose:

```c
int fact(int n) {
    int temp;
    if(n == 0)
        return 1;

    temp = fact(n-1);
    return n * temp;
}
```

At deepest point:

```text
fact(0)
```

stack could be shown as:

```text
┌────────────────┐
│ fact(0)        │
│ n = 0          │
│ temp = --      │
│ return addr    │
│ control→fact1  │
├────────────────┤
│ fact(1)        │
│ n = 1          │
│ temp = waiting │
│ control→fact2  │
├────────────────┤
│ fact(2)        │
│ n = 2          │
│ temp = waiting │
│ control→fact3  │
├────────────────┤
│ fact(3)        │
│ n = 3          │
│ temp = waiting │
│ control→main   │
├────────────────┤
│ main           │
└────────────────┘
```

As returns happen:

```text
fact(1).temp = 1
fact(2).temp = 1
fact(3).temp = 2
...
```

For any exam question asking:

> “Show activation records with their values”

put in:

```text
parameters
locals
return value where known
control link
```

You usually don't need to invent machine-level details unless explicitly requested.

---

# 16. ACTIVATION TREE WITH BRANCHING: FIBONACCI

Factorial makes a straight chain.

Fibonacci makes an actual tree.

```c
int f(int n) {
    int s, t;

    if(n < 2)
        return 1;

    s = f(n-1);
    t = f(n-2);

    return s+t;
}
```

For:

```text
f(4)
```

activation tree:

```text
                 f(4)
               /      \
            f(3)       f(2)
           /   \       /   \
        f(2)   f(1)  f(1)  f(0)
        /  \
     f(1)  f(0)
```

If called from `main`:

```text
main
 |
f(4)
...
```

### Critical rule

Execute left child completely first because code says:

```c
s = f(n-1);   // first
t = f(n-2);   // second
```

So execution follows **depth-first, left-to-right**.

That depth-first relationship is stated explicitly in the lecture. 

---

# 17. HARD FIBONACCI EXAM QUESTION MADE EASY

A historical question gives recursive Fibonacci and asks something like:

> Show the stack when `f(1)` is about to return for the first time / fifth time.

This type appears in the exam set. 

Do NOT simulate everything randomly.

### Method

1. Draw activation tree.
2. Walk it depth-first left-to-right.
3. Number each `f(1)` as encountered.
4. Find required occurrence.
5. Stack = path from that `f(1)` to root.

Example for `f(5)`:

First `f(1)` occurs here:

```text
main
 |
f(5)
 |
f(4)
 |
f(3)
 |
f(2)
 |
f(1)  ← first
```

So stack:

```text
f(1)
f(2)
f(3)
f(4)
f(5)
main
```

For the fifth `f(1)`, continue the DFS count.

At that instant the stack is simply whatever ancestor path leads to that fifth node.

No special trick.

---

# 18. EVENT-LIST ACTIVATION TREE QUESTIONS

Sometimes the exam doesn't give code.

Instead:

```text
enter main
enter A
enter B
leave B
enter C
leave C
leave A
leave main
```

They ask:

> Construct activation tree.

This is easy.

Process sequentially.

Start:

```text
enter main
```

Tree:

```text
main
```

`enter A`:

```text
main
 |
 A
```

`enter B`:

```text
main
 |
 A
 |
 B
```

`leave B`:

Go back to `A`.

`enter C`:

```text
main
 |
 A
/ \
B  C
```

Done.

### Rule

```text
ENTER X → create child of currently active function

LEAVE X → return to its parent
```

2023 contains this exact style of activation-tree question using `enter` and `leave` events. 

---

# 19. CALLING SEQUENCE

Now ask:

> What exactly happens when A calls B?

The slides call this the **calling sequence**. 

Example:

```c
x = add(10,20);
```

Before executing `add`, runtime must prepare.

Think:

```text
CALLER = main
CALLEE = add
```

---

# 20. CALLER VS CALLEE

### Caller

The function making the call.

### Callee

The function being called.

Example:

```c
main() {
    add(10,20);
}
```

```text
main = caller
add  = callee
```

---

# 21. CALL SEQUENCE — SUPER EASY VERSION

The slides divide responsibility between caller and callee. 

Think:

### Caller prepares the package

```text
1. evaluate arguments
2. save where execution should return
3. prepare/move stack
```

### Callee settles in

```text
4. save registers/status
5. create/init local data
6. start execution
```

Mental picture:

> Going to a hotel.

Caller makes booking and gives information.

Callee enters room, stores belongings, starts work.

---

# 22. RETURN SEQUENCE

When function finishes:

```text
1. produce return value
2. restore machine/register state
3. restore stack
4. jump to return address
5. caller receives result
```

The slides give essentially this exact sequence. 

### Memory trick

```text
CALL:
prepare → push → enter

RETURN:
answer → restore → pop → go back
```

2024 directly asks for the calling and return sequences of nested functions. 

---

# 23. WHAT IS THE RETURN ADDRESS?

Look:

```c
main() {
    x = 5;
    y = add(2,3);
    z = 9;
}
```

When CPU jumps into `add()`, it must remember:

> After `add` finishes, continue at `z = 9` / the instruction after the call.

That location is the:

# **Return Address**

Think bookmark.

```text
CALL add
↓
bookmark next instruction
↓
execute add
↓
RETURN
↓
jump to bookmark
```

---

# 24. PARAMETER PASSING

The runtime slide includes:

* pass by value
* pass by reference
* pass by name

and explains value/reference directly. 

For this exam set, value/reference are the useful ones.

---

# 25. CALL BY VALUE

```c
void change(int x) {
    x = 100;
}

int a = 5;
change(a);
```

Function receives a **copy**:

```text
original a = 5

copy x = 5
```

then:

```text
x = 100
```

Original:

```text
a = 5
```

unchanged.

Mental image:

> I photocopy your paper and write on my copy.

---

# 26. CALL BY REFERENCE

Function receives access/address to original value.

```text
a = 5
```

Function changes referenced value to:

```text
100
```

Now:

```text
a = 100
```

Mental image:

> You give me your actual notebook, not a photocopy.

### Easy comparison

| Value                | Reference                |
| -------------------- | ------------------------ |
| copy passed          | address/reference passed |
| original safe        | original can change      |
| no extra indirection | extra indirection needed |

---

# 27. CALL BY NAME

The slide lists this mapping convention but does not develop it further. 

For this chapter's exam preparation, just recognize the name:

> **Pass-by-name delays evaluation/use of the actual argument and behaves roughly like substituting the actual expression when needed.**

Do not spend serious time on it unless a new question specifically asks.

---

# 28. VARIABLE-LENGTH DATA ON STACK

This slide can look terrifying. It isn't.

Normally compiler knows sizes:

```c
int x;
double y;
```

So it can say:

```text
x is offset +4
y is offset +8
```

Easy.

But suppose array size is known only at runtime:

```c
int a[n];
```

Then compiler doesn't know its exact size beforehand.

Solution idea:

```text
fixed part of AR:
pointer to a

variable-size area:
actual array a
```

So:

```text
Activation Record
┌─────────────────┐
│ pointer to a ─────────────┐
│ pointer to b ────────┐    │
│ fixed fields          │    │
└─────────────────┘     │    │
                        ↓    ↓
                  ┌──────────────┐
                  │ array b      │
                  │ array a      │
                  └──────────────┘
```

The lecture specifically introduces variable-length data on the stack. 

For exam purposes, understand **why pointers are needed**. You don't need PhD-level stack-layout details.

---

# 29. NON-LOCAL VARIABLES — START FROM ZERO

Consider:

```c
void outer() {
    int x = 10;

    void inner() {
        printf("%d", x);
    }

    inner();
}
```

`x` is not local to `inner`.

But `inner` is nested inside `outer`, so it is allowed to use it.

For `inner`:

```text
x = NON-LOCAL VARIABLE
```

Runtime problem:

> `inner` has its own activation record.
> So how does it find `outer`'s activation record containing `x`?

Answer:

# **Access Link**

---

# 30. ACCESS LINK

The slides say:

> If `p` is nested immediately inside `q`, the access link of `p` points to the most recent activation of `q`. 

Example:

```text
outer()
   |
   └── inner()
```

Runtime:

```text
┌───────────────┐
│ AR inner      │
│ access link ───────────┐
└───────────────┘         │
                          ↓
                  ┌──────────────┐
                  │ AR outer     │
                  │ x = 10       │
                  └──────────────┘
```

Then `inner` follows the link and gets `x`.

---

# 31. MULTIPLE NESTING LEVELS

```text
A
└── B
    └── C
```

Suppose `C` needs a variable from `A`.

Access-link chain:

```text
C → B → A
```

If variable is in B:

```text
C → B
```

If variable is local to C:

```text
no link needed
```

Mental picture:

> Access links are stairs going outward through nested scopes.

---

# 32. PROCEDURE NESTING DEPTH

The slides define nesting depth directly. 

Example:

```text
A              depth 1
 └─ B          depth 2
     └─ C      depth 3
```

So:

```text
A = 1
B = 2
C = 3
```

A procedure nested inside a depth `i` procedure has:

```text
depth = i + 1
```

---

# 33. LIMITATION OF ACCESS LINKS

Suppose:

```text
depth 1: A
depth 2: B
depth 3: C
depth 4: D
depth 5: E
```

`E` wants A's variable.

Needs:

```text
E → D → C → B → A
```

Five levels means lots of pointer following.

This is the weakness:

> **Lookup time increases with number of nesting levels.**

The slides compare access links and displays and explicitly note that long access-link chains can become costly. 

Solution:

# **Display**

---

# 34. DISPLAY — THE EASY WAY

A display is basically an array:

```text
d[1]
d[2]
d[3]
...
```

Each entry points directly to the **most recent activation at that nesting level**. 

Suppose:

```text
A depth 1
B depth 2
C depth 3
```

Display:

```text
d[1] → A's activation record
d[2] → B's activation record
d[3] → C's activation record
```

Now C needs variable from A.

With access links:

```text
C → B → A
```

With display:

```text
d[1] → A
```

Direct hit.

---

# 35. ACCESS LINK VS DISPLAY

This table is exam gold:

| Access Link              | Display                         |
| ------------------------ | ------------------------------- |
| chain of pointers        | global array of pointers        |
| follow links upward      | direct pointer by nesting depth |
| lookup cost varies       | lookup cost constant            |
| long chains can be slow  | fast non-local access           |
| simpler mental structure | extra display maintenance       |

The slides explicitly make this cost comparison. 

Memory trick:

```text
ACCESS LINK = staircase

DISPLAY = elevator
```

Staircase:

```text
5 → 4 → 3 → 2 → 1
```

Elevator:

```text
press 1
```

You will never forget this.

---

# 36. 2024-TYPE ACCESS LINK QUESTION

Consider:

```c
int base = 10;

void outer() {
    int offset = 5;

    void inner() {
        int result = base + offset;
    }

    inner();
}
```

This style appears in the 2024 paper. 

Understand variable locations:

```text
base
→ global
→ static area

offset
→ local to outer
→ AR of outer

result
→ local to inner
→ AR of inner
```

During `inner()`:

```text
AR inner
  access link
      ↓
AR outer
  offset = 5
```

`base` does not require walking through outer's frame because it is global/static.

### Display version

Using slide convention where top-level function depth = 1:

```text
d[1] → AR(outer)
d[2] → AR(inner)
```

Then `inner` gets `offset` through:

```text
d[1]
```

---

# 37. CONTROL LINK VS ACCESS LINK WITH SAME EXAMPLE

Suppose:

```text
outer calls inner
```

Then conveniently both may point toward outer:

```text
inner.control_link → outer
inner.access_link  → outer
```

But **do not conclude they are the same thing.**

Reasons are different:

```text
control link:
outer CALLED inner

access link:
outer LEXICALLY CONTAINS inner
```

In more complicated programs, those may point to different activations.

---

# 38. DISPLAY UPDATES DURING RECURSION

This can look confusing in slide diagrams.

Suppose:

```text
A depth 1
B depth 2
```

Current:

```text
d[1] → A
d[2] → B1
```

Then B recursively calls B again:

```text
B1 → B2
```

Now the most recent depth-2 activation is B2.

So:

```text
save old d[2] = B1

d[2] → B2
```

When B2 returns:

```text
restore d[2] → B1
```

That's why activation records may contain a **saved display value** in display implementations.

The slide diagrams on pages 36–38 show exactly this save/restore behavior. 

---

# 39. HEAP MANAGEMENT

Now leave functions for a moment.

Heap handles objects whose lifetime doesn't neatly follow function-call stack order.

```c
Node *p = new Node();
```

Heap manager must do two basic jobs:

```text
ALLOCATE
give memory

DEALLOCATE
take memory back
```

The runtime slides describe these two operations directly. 

---

# 40. WHY HEAP IS HARDER THAN STACK

Stack:

```text
push
push
pop
pop
```

Very organized.

Heap:

```text
allocate 20 bytes
allocate 50
free first object
allocate 10
free third object
allocate 80
...
```

Random lifetimes.

So heap can become:

```text
[USED][FREE][USED][FREE][FREE][USED]
```

That creates:

# Fragmentation

Meaning:

> Free memory exists, but it is broken into separated pieces.

---

# 41. MEMORY MANAGER GOALS

The slides emphasize:

```text
1. Space efficiency
2. Program efficiency
3. Low overhead
```



Plain English:

```text
Don't waste memory.
Don't make program slow.
Don't spend forever managing memory.
```

---

# 42. FIRST-FIT ALLOCATION

Suppose free spaces are:

```text
10   40   25   60
```

Need:

```text
20
```

First-fit scans from beginning:

```text
10 → too small
40 → enough ✅
```

Use the `40`.

The slides describe first-fit as traversing the free list until a sufficiently large block is found. 

If block is larger than needed:

```text
40 needed
20 requested
```

split:

```text
20 USED
20 FREE
```

---

# 43. REDUCING FRAGMENTATION

Suppose:

```text
[FREE 20][FREE 30]
```

Instead of keeping two blocks:

```text
20 + 30
```

merge:

```text
FREE 50
```

The slides call this merging free blocks and also mention best-fit/next-fit variants. 

For your exam focus, just understand the idea.

---

# 44. MANUAL DEALLOCATION PROBLEMS

Suppose:

```c
p = new Object();
```

and you never free it even though nothing can use it anymore.

That is:

# **Memory Leak**

Now suppose:

```c
delete p;
```

but another pointer still tries to use the deleted object.

That is:

# **Dangling Pointer**

The runtime chapter explicitly names both problems. 

Memory trick:

```text
LEAK
memory exists but you lost useful access

DANGLING
pointer exists but object is gone
```

---

# 45. GARBAGE

Suppose:

```c
Object *p = new Object();
p = NULL;
```

The allocated object still exists in heap.

But no usable pointer reaches it.

That object is:

# **Garbage**

The slides define unreachable data as garbage and introduce automatic garbage collection. 

---

# 46. GARBAGE COLLECTION

Garbage collection means:

> Automatically reclaim heap objects that are no longer reachable.

So programmer does not manually need to delete every dead object.

For your slide-based syllabus, focus especially on:

# **Reachability + Reference Counting**

---

# 47. REACHABILITY

This sounds scary but is ridiculously simple.

Suppose:

```text
X → A → B → C
```

If program has direct access to `X`, then:

```text
X reachable
A reachable
B reachable
C reachable
```

Now suppose:

```text
D → E
```

but nothing reachable points to D.

Then:

```text
D unreachable
E unreachable
```

= garbage.

---

# 48. ROOT SET

Where does reachability start?

From objects/references the program can directly access.

That starting group is called the:

# **Root Set**

The slides define root set and recursively reachable objects. 

Mental picture:

```text
ROOTS
 ↓
reachable object
 ↓
reachable object
 ↓
reachable object
```

Anything not connected to the root world:

```text
GARBAGE
```

---

# 49. REACHABILITY GRAPH

Picture:

```text
ROOT
 |
 A
/ \
B  C
|   \
D    E
```

Everything reachable from ROOT stays alive.

But:

```text
F → G
```

with no path from root:

```text
F, G = garbage
```

---

# 50. THINGS THAT CHANGE REACHABILITY

The slide gives four main operations. 

### Object allocation

```c
p = new Object();
```

new object appears.

### Parameter passing / return

References move between functions.

### Reference assignment

```c
u = v;
```

`u` stops pointing to old object and points to v's object.

### Procedure return

Local references disappear when the frame disappears.

That's enough to solve most object-network questions.

---

# 51. REFERENCE COUNTING — EASIEST POSSIBLE EXPLANATION

Every object stores:

```text
How many references currently point to me?
```

That number is its:

# **Reference Count**

Example:

```text
p → A
```

Then:

```text
A.count = 1
```

Now:

```text
q → A
```

Then:

```text
A.count = 2
```

Now:

```text
p = NULL
```

Then:

```text
A.count = 1
```

Now:

```text
q = NULL
```

Then:

```text
A.count = 0
```

So A becomes garbage and may be deallocated.

The reference-counting slide states that an object may be deleted when its count drops to zero. 

---

# 52. REFERENCE COUNTING RULES

Use these rules in every exam calculation:

```text
new pointer points to object
→ count +1

pointer stops pointing to object
→ count -1

pointer changes A → B
→ A -1
→ B +1

count becomes 0
→ garbage
```

That's basically the entire algorithm you need.

---

# 53. EXAMPLE — REFERENCE ASSIGNMENT

Start:

```text
p → A
q → B
```

Counts:

```text
A = 1
B = 1
```

Execute:

```c
p = q;
```

Before:

```text
p → A
q → B
```

After:

```text
p ─┐
   ↓
   B
   ↑
q ─┘
```

Counts:

```text
A: 1 → 0   ← garbage
B: 1 → 2
```

This pattern is extremely important for exam problems.

---

# 54. 2023-TYPE GARBAGE QUESTION

Consider:

```c
Object *obj1 = new Object();   // A
Object *obj2 = new Object();   // B

obj1 = obj2;
```

Before reassignment:

```text
obj1 → A    count A = 1
obj2 → B    count B = 1
```

After:

```text
obj1 ─┐
      ↓
      B
      ↑
obj2 ─┘
```

Now:

```text
A count = 0 → GARBAGE
B count = 2
```

That is the core move required by the heap/reference-counting style seen in the recent papers. 

---

# 55. REFERENCE COUNT TABLE METHOD

Whenever given code like:

```c
p = new;
q = new;
r = p;
q = p;
r = q;
...
```

Do not draw everything repeatedly.

Make:

| Step    | Object A | Object B | Garbage |
| ------- | -------: | -------: | ------- |
| p=new A |        1 |        0 | —       |
| q=new B |        1 |        1 | —       |
| r=p     |        2 |        1 | —       |
| p=q     |        1 |        2 | —       |
| r=q     |        0 |        3 | A       |

Instant full solution.

---

# 56. STATIC / STACK / HEAP QUESTION TEMPLATE

Historical papers repeatedly ask comparison of storage strategies; e.g. the 2019 runtime section asks activation record/tree and comparison among static, stack and heap allocation. 

For exam:

| Point                   | Static        | Stack              | Heap                     |
| ----------------------- | ------------- | ------------------ | ------------------------ |
| Allocation time         | compile time  | runtime            | runtime                  |
| Managed as              | fixed         | LIFO               | arbitrary                |
| Typical data            | static/global | AR + locals        | dynamic objects          |
| Allocation/deallocation | fixed         | automatic          | manual/automatic         |
| Recursion               | not suitable  | suitable           | possible                 |
| Cost                    | low           | low                | higher                   |
| Main limitation         | inflexible    | LIFO lifetime only | fragmentation/management |

That table is enough for a 4-mark comparison.

---

# 57. ACTIVATION RECORD THEORY QUESTION TEMPLATE

Question:

> Define activation record. Mention contents.

Write:

> An activation record or stack frame stores information required for one live procedure activation on the control stack.

Then draw:

```text
┌───────────────────┐
│ Actual parameters │
│ Returned value    │
│ Control link      │
│ Access link       │
│ Saved status      │
│ Local data        │
│ Temporaries       │
└───────────────────┘
```

Then 1 short purpose each.

The seven-field structure appears explicitly in the lecture. 

That is enough for full marks on the common “seven fields” question.

---

# 58. ACTIVATION TREE QUESTION TEMPLATE

Given code:

```c
main(){
   A();
   B();
}

A(){
   C();
}
```

Do:

```text
        main
       /    \
      A      B
      |
      C
```

Then say:

```text
Root = main
Parent = caller
Child = called activation
Execution = depth-first left-to-right
```

Done.

---

# 59. STACK GROW/SHRINK QUESTION TEMPLATE

Given recursion:

```c
fact(3)
```

Write calls:

```text
fact3 → fact2 → fact1 → fact0
```

Then show:

```text
Grow:

fact0
fact1
fact2
fact3
main
```

Then returns:

```text
fact0 pops
fact1 pops
fact2 pops
fact3 pops
```

If asked for values, insert parameters/locals into each frame.

---

# 60. “STACK AT A PARTICULAR MOMENT” TEMPLATE

This is the most important solving shortcut.

Question says:

> What does stack look like when function X is about to return?

Do:

```text
1. Build activation tree.
2. Find that exact X activation.
3. Trace X → parent → parent → ... → main.
4. Reverse visually: X is top, main is bottom.
```

Never simulate random push/pop if you already have the tree.

---

# 61. ACCESS LINK QUESTION TEMPLATE

Given nested functions:

```text
A
└── B
    └── C
```

Suppose C accesses variable from A.

Draw:

```text
AR C
 │ access link
 ↓
AR B
 │ access link
 ↓
AR A
```

Then:

> C follows two access links to reach A's activation record.

Limitation:

> Long nesting means multiple pointer traversals.

Solution:

> Use display.

---

# 62. DISPLAY QUESTION TEMPLATE

If:

```text
A depth 1
B depth 2
C depth 3
```

During C:

```text
d[1] → AR(A)
d[2] → AR(B)
d[3] → AR(C)
```

If C wants A's variable:

```text
follow d[1]
```

Then known offset inside A's frame.

The slides explain this direct lookup explicitly. 

---

# 63. CALL/RETURN SEQUENCE QUESTION TEMPLATE

### Call

```text
Caller:
1. evaluate parameters
2. save return address / old stack info
3. adjust stack

Callee:
4. save registers/status
5. initialize locals
6. execute
```

### Return

```text
Callee:
1. place return value
2. restore stack/registers
3. jump to saved return address

Caller:
4. receives return value
```

This is almost directly the lecture sequence. 

---

# 64. “ASSEMBLY USING STACK ALLOCATION + AR” QUESTIONS

You will see questions like:

> Generate assembly code using stack allocation and activation record.

Runtime knowledge tells you **what must happen around a call**:

```text
save parameters
save return address
create/push frame
execute callee
store return value
restore state
pop frame
continue caller
```

Exact `LD`, `ST`, `ADD`, register choice and numeric instruction-address calculation belong to the **Code Generation** portion of your course, not Runtime Environment itself.

So mentally split the question:

```text
FUNCTION/STACK PART
→ Runtime Environment

LD/ST/ADD/register-generation part
→ Code Generation
```

The 2024 paper mixes these two areas in exactly this way. 

---

# 65. THE ENTIRE CHAPTER AS ONE STORY

Now connect everything.

You write:

```c
int main() {
    A();
}
```

Program starts.

### 1. Memory exists as:

```text
Code
Static
Heap
Stack
```

### 2. `main` begins

Runtime creates:

```text
AR(main)
```

on stack.

### 3. `main` calls A

Create:

```text
AR(A)
```

with:

```text
parameters
return value area
control link
access link if needed
saved machine state
locals
temporaries
```

### 4. If A calls B

Push B.

### 5. If B returns

Pop B.

### 6. If recursive

Multiple ARs of same function exist simultaneously.

### 7. Whole call history can be shown as activation tree

Current live path is control stack.

### 8. Nested procedure needs outer variable

Use access link.

### 9. Too many access-link jumps

Use display.

### 10. Object created dynamically

Store it in heap.

### 11. Object becomes unreachable

It becomes garbage.

### 12. Reference count becomes zero

Reference-counting collector can reclaim it.

That is basically the whole Runtime Environment chapter.

---

# 66. WHAT QUESTIONS HAVE ACTUALLY BEEN TARGETING

Ignoring the five legacy concepts as requested, the recurring question families across the papers reduce to these:

| Question family                   | What you need      |
| --------------------------------- | ------------------ |
| Define runtime environment/issues | Section 0–1        |
| Compare static/stack/heap         | Section 2          |
| Define activation                 | Section 3          |
| Draw activation tree              | Sections 5, 16, 18 |
| Explain control stack             | Sections 6–7       |
| Define/draw activation record     | Sections 8–10      |
| Explain 7 AR fields               | Section 9          |
| Recursive stack grow/shrink       | Sections 12–15     |
| Show AR values                    | Sections 15–17     |
| Calling/return sequence           | Sections 19–22     |
| Caller/callee responsibility      | Sections 20–22     |
| Parameter passing                 | Sections 24–27     |
| Access non-local variable         | Sections 29–33     |
| Access link limitation            | Section 33         |
| Display implementation            | Sections 34–38     |
| Access link vs display            | Section 35         |
| Heap management                   | Sections 39–43     |
| Garbage/memory leak/dangling      | Sections 44–45     |
| Reachability                      | Sections 46–50     |
| Reference counting                | Sections 51–55     |

That is the runtime syllabus you should teach.

---

# 67. TEACHING ORDER I WOULD USE WITH ZERO-BASIC STUDENTS

Do **not** start with activation records.

Start like this:

```text
1. What happens while a program runs?
        ↓
2. Code / Static / Heap / Stack
        ↓
3. Static vs Stack vs Heap
        ↓
4. What is a function call / activation?
        ↓
5. Activation tree
        ↓
6. Control stack
        ↓
7. Activation record
        ↓
8. Recursive stack growth/shrink
        ↓
9. Calling & return sequence
        ↓
10. Parameter passing
        ↓
11. Nested function/non-local variable problem
        ↓
12. Access link
        ↓
13. Display
        ↓
14. Heap management
        ↓
15. Garbage + reachability
        ↓
16. Reference counting
```

Why this order?

Because every later topic uses an earlier mental picture.

Do it in the opposite order and students memorize without understanding.

---

# 68. THE FIVE SENTENCES STUDENTS MUST NEVER FORGET

If they forget everything else, these rebuild the chapter:

> **1. Runtime Environment manages memory and procedure execution while the program runs.**

> **2. Every live function call has one activation record on the stack.**

> **3. Activation tree shows all calls; control stack shows only currently alive calls.**

> **4. Control link points to caller; access link points toward the lexically enclosing activation for non-local data.**

> **5. Heap stores dynamic objects; unreachable objects are garbage.**

From these five sentences, almost the whole chapter can be reconstructed.

---

# 69. 30-SECOND MENTAL MAP

```text
                RUNTIME ENVIRONMENT
                        |
        ┌───────────────┴──────────────┐
        |                              |
     STORAGE                       PROCEDURES
        |                              |
 Static / Stack / Heap        Call = Activation
        |                              |
        |                       Activation Tree
        |                              |
        |                       Control Stack
        |                              |
        |                     Activation Record
        |                              |
        |                ┌─────────────┴─────────────┐
        |                |                           |
        |          Call / Return              Non-local access
        |                                            |
        |                                     Access Link
        |                                            |
        |                                         Display
        |
       HEAP
        |
 Allocation / Deallocation
        |
     Garbage
        |
   Reachability
        |
Reference Counting
```

This is the entire chapter on one screen.

---

# 70. FINAL EXAM MEMORY SHEET

### Runtime

```text
Manages storage + calls + parameters + variable access while program runs
```

### Memory

```text
Code | Static | Heap | Stack
```

### Static

```text
fixed before runtime
no recursion
```

### Stack

```text
function calls
push on call
pop on return
```

### Heap

```text
dynamic arbitrary-lifetime data
```

### Activation

```text
one execution of a procedure
```

### Activation Tree

```text
root = main
parent = caller
DFS execution
```

### Control Stack

```text
currently live path of activation tree
```

### Activation Record

```text
Actual parameters
Return value
Control link
Access link
Saved status
Local data
Temporaries
```

### Links

```text
Control → caller
Access  → enclosing scope
```

### Call/return

```text
CALL = prepare + push + execute
RETURN = result + restore + pop + continue
```

### Access link

```text
walk outer scopes
```

### Display

```text
d[level] → latest AR at that nesting level
```

### Heap problems

```text
memory leak
dangling pointer
fragmentation
```

### Garbage

```text
unreachable object
```

### Reference counting

```text
new reference → +1
lost reference → -1
0 → garbage
```

---

## The main thing to understand

Runtime Environment is **not 20 disconnected definitions**.

It is one story:

```text
A program runs
→ functions start and end
→ every live function needs memory
→ stack stores their frames
→ frames form the current call chain
→ all calls form an activation tree
→ frames contain parameters, links, locals, return info
→ access links/displays find outer variables
→ heap stores dynamic objects
→ unreachable heap objects become garbage
→ reference counting can identify them
```

If you can teach **that story** first, then every diagram, theory question, recursive stack problem and access-link/display problem becomes much easier instead of feeling like a new chapter every time.

**This is the missing module.** After this, the pure Runtime questions + the Runtime/assembly mixed questions in your collection are covered conceptually. The only thing I will **not invent** is an exact numeric byte address when the question does not provide instruction sizes/frame sizes; your slides themselves use symbolic `recordSize` for that. The important rule is that the saved return address is the instruction immediately after the call. 

# Final Module: Assembly Using Stack Allocation + Activation Record

## 1. First: WTF is this question asking?

You already know this:

```text
main calls inc
      ↓
AR(main)
AR(inc)
```

But now examiner says:

> “Show assembly code using **stack allocation and activation record**.”

They want you to show **how the machine actually performs that call**:

```text
Before call:
SP → AR(main)

        CALL inc

During inc:
SP → AR(inc)
     AR(main)

        RETURN

After return:
SP → AR(main)
```

So this question is simply:

> **Normal assembly + move SP to make a new activation record + save return address + call + return + restore SP.**

The Code Generation slide uses exactly this mechanism. 

---

# 2. Only assembly instructions you need

Your target machine is a simple three-address register machine with load/store, arithmetic and branch instructions. 

### `LD` = get something

```text
LD R1, x
```

means:

```text
R1 = x
```

```text
LD R1, #10
```

means:

```text
R1 = 10
```

The `#` is important:

```text
#10 = literal number 10
10   = location/address 10
```

The slides explicitly use `#` for immediate constants. 

---

### `ST` = put something into memory

```text
ST x, R1
```

means:

```text
x = R1
```

So memorize:

```text
LD → memory → register

ST → register → memory
```

The course target machine defines exactly these operations. 

---

### `ADD / SUB`

```text
ADD R1, R2, R3
```

means:

```text
R1 = R2 + R3
```

and:

```text
SUB R1, R2, R3
```

means:

```text
R1 = R2 - R3
```

You can also use constants:

```text
ADD R1, R1, #1
```

means:

```text
R1 = R1 + 1
```



---

### `BR`

```text
BR L
```

means:

```text
jump to L
```

And conditional branch:

```text
BLTZ R1, L
```

means:

```text
if R1 < 0
    jump L
```



Therefore:

```c
if(x < y)
```

can become:

```text
LD   R1, x
LD   R2, y
SUB  R1, R1, R2
BLTZ R1, TRUE
```

This is also the exact pattern shown in your slide. 

---

# 3. Understand these weird symbols

This is where the assembly starts looking scary.

### `SP`

`SP` = **Stack Pointer**

Think:

```text
SP = "Where is the current activation record?"
```

Before calling `inc`:

```text
SP
 ↓
┌────────────┐
│ AR(main)   │
└────────────┘
```

During `inc`:

```text
        SP
         ↓
┌────────────┐
│ AR(inc)    │
├────────────┤
│ AR(main)   │
└────────────┘
```

---

### `0(SP)`

Means:

> memory at `SP + 0`

Basically the beginning of the current activation record.

Your stack-allocation slide uses this location to hold the **return address**.

```text
0(SP) = return address
```

---

### `4(SP)`, `8(SP)`, etc.

Conceptually:

```text
0(SP)       return address
argOff(SP)  parameter
retOff(SP)  returned value
localOff(SP) local variable
```

**Important:** your supplied slides do not establish one universal numeric offset such as “parameter is always 4(SP).” Activation-record field positions can vary by implementation. So in a solution, use the offsets supplied by the problem/teacher, or symbolic offsets if none are given. The Runtime slides explicitly say activation-record field contents/positions may vary. 

---

### `*0(SP)`

This is the fun one.

Suppose:

```text
0(SP) = 152
```

Then:

```text
BR *0(SP)
```

means:

```text
jump to address 152
```

In plain English:

> **Go back to wherever the caller told me to return.**

---

# 4. The ONE procedure-call pattern to memorize

Forget every individual past question for a moment.

This is the skeleton behind ALL of them:

```text
CALLER:

ADD SP, SP, #caller.recordSize
ST  *SP, #RETURN
BR  CALLEE

RETURN:
SUB SP, SP, #caller.recordSize
```

And callee ends with:

```text
CALLEE:
   ...
   BR *0(SP)
```

Your lecture literally gives this stack-allocation pattern: increase `SP`, store the return address, branch to the callee; the callee returns through `*0(SP)`, and the caller restores `SP`. 

### Translate it into human language

```text
ADD SP...
     ↓
make room for callee's AR

ST *SP,#RETURN
     ↓
write "come back here" into callee's AR

BR CALLEE
     ↓
go execute function

BR *0(SP)
     ↓
callee reads saved address and comes back

SUB SP...
     ↓
delete/pop callee AR
```

That's **the whole trick**.

---

# 5. Why `ADD SP` means PUSH here

Normally you may imagine a stack growing downward.

Forget CPU-specific ideas here.

**In this course's abstract target machine**, the slide implements stack growth using:

```text
ADD SP, SP, #caller.recordSize
```

So follow the lecturer's convention.

If:

```text
SP = 600
main.recordSize = 40
```

then:

```text
ADD SP,SP,#40
```

gives:

```text
SP = 640
```

Now `640` is the start of the next activation record.

Return:

```text
SUB SP,SP,#40
```

gives:

```text
SP = 600
```

Back to `main`.

---

# 6. The slide's own example decoded

The slide gives:

```text
100: LD  SP, #600
...
128: ADD SP, SP, #msize
136: ST  *SP, #152
144: BR  300
152: SUB SP, SP, #msize
```



Ignore the numbers for one second.

It's simply:

```text
LD SP,#600
```

Start stack at 600.

Then:

```text
ADD SP,SP,#msize
```

Create next frame.

Then:

```text
ST *SP,#152
```

Tell callee:

> “When you're done, come back to 152.”

Then:

```text
BR 300
```

300 = callee's code.

Callee eventually:

```text
BR *0(SP)
```

It reads:

```text
0(SP) = 152
```

so jumps to `152`.

At 152:

```text
SUB SP,SP,#msize
```

Remove callee frame.

### The most important relationship

```text
136: ST *SP,#152
144: BR 300
152: ...
```

Why `152`?

Because **152 is where execution must continue after the call**.

So:

# `Return Address = instruction after the call`

---

# 7. Add parameters to the picture

Suppose:

```c
i = inc(i);
```

`inc` needs the value of `i`.

Its AR conceptually contains:

```text
AR(inc)

┌──────────────────────┐
│ return address       │ ← 0(SP)
├──────────────────────┤
│ parameter x          │
├──────────────────────┤
│ returned value       │
├──────────────────────┤
│ ...other AR fields   │
└──────────────────────┘
```

The general activation record in your Runtime slides includes actual parameters and returned values along with links, saved status, locals and temporaries. 

We'll call the unspecified offsets:

```text
ARG(SP)
RET(SP)
```

Not because `ARG` is a real machine instruction.

It means:

```text
ARG = whatever offset has been assigned
      to the argument field
```

---

# 8. How `i = inc(i)` happens

Suppose currently:

```text
i = 4
```

### Before call

```text
SP → AR(main)

i = 4
```

### Step A: get argument

```text
LD R1, i
```

Now:

```text
R1 = 4
```

### Step B: make inc frame

```text
ADD SP, SP, #main.recordSize
```

Now:

```text
SP → AR(inc)
```

### Step C: put argument inside inc frame

```text
ST ARG(SP), R1
```

Now:

```text
x = 4
```

### Step D: store return address

```text
ST 0(SP), #BACK
```

### Step E: call

```text
BR INC
```

---

# 9. Inside `inc`

Source:

```c
int inc(int x) {
    x = x + 1;
    return x;
}
```

Assembly idea:

```text
INC:
    LD   R1, ARG(SP)
    ADD  R1, R1, #1
    ST   ARG(SP), R1
    ST   RET(SP), R1
    BR   *0(SP)
```

Human translation:

```text
get x
x = x+1
save changed x
save answer
return
```

That's it.

---

# 10. Back inside caller

When:

```text
BR *0(SP)
```

runs, we reach:

```text
BACK:
```

Notice:

**SP still points to the callee frame right now.**

So first obtain its returned value:

```text
LD R1, RET(SP)
```

Then remove the frame:

```text
SUB SP, SP, #main.recordSize
```

Now:

```text
SP → AR(main)
```

Then save result:

```text
ST i(SP), R1
```

So:

```text
i = returned result
```

---

# 11. Full 2024-style problem

Your 2024 Q5(b) uses essentially:

```c
int main() {
    int i = 0;

    while(i < 10) {
        i = inc(i);
    }

    return i;
}

int inc(int x) {
    x = x + 1;
    return x;
}
```

and asks for assembly using stack allocation and activation record, with `main` starting at address 100. 

Here is how to think about it.

## First ignore function call completely

Translate:

```c
i = 0;
```

to:

```text
LD R1,#0
ST i(SP),R1
```

---

## Translate `i < 10`

We need:

```text
i - 10 < 0
```

So:

```text
TEST:
    LD   R1, i(SP)
    SUB  R1, R1, #10
    BLTZ R1, CALL_INC
    BR   END
```

No mystery.

---

## Then call `inc`

```text
CALL_INC:
    LD   R1, i(SP)               ; get actual parameter

    ADD  SP, SP, #main.recordSize
    ST   ARG(SP), R1             ; x = i
    ST   0(SP), #BACK_FROM_INC   ; return address
    BR   INC
```

---

## Come back

```text
BACK_FROM_INC:
    LD   R1, RET(SP)             ; result from inc
    SUB  SP, SP, #main.recordSize
    ST   i(SP), R1               ; i = returned result
    BR   TEST
```

---

## End

```text
END:
    LD R1, i(SP)
    HALT
```

---

## `inc`

```text
INC:
    LD   R1, ARG(SP)
    ADD  R1, R1, #1
    ST   ARG(SP), R1
    ST   RET(SP), R1
    BR   *0(SP)
```

### Starting stack

At beginning:

```text
100: LD SP,#stackStart
```

because the question says main begins at address `100`.

After that, code continues.

**Do not invent exact later numeric addresses unless your instruction sizes are known.** The lecture's own worked target-code example can use numbers such as `128`, `136`, `144`, `152` because that particular example already fixes the generated-code layout. 

If numerical addresses are supplied/derivable in your exam:

```text
ST 0(SP), #address_after_BR
```

That's the only return-address rule you need.

---

# 12. See the stack while the above program runs

At start:

```text
SP
 ↓
┌─────────────┐
│ main        │
│ i = 0       │
└─────────────┘
```

Call `inc(0)`:

```text
SP
 ↓
┌─────────────┐
│ inc         │
│ x = 0       │
│ return addr │
├─────────────┤
│ main        │
│ i = 0       │
└─────────────┘
```

`inc` returns `1`.

Pop:

```text
SP
 ↓
┌─────────────┐
│ main        │
│ i = 1       │
└─────────────┘
```

Then:

```text
inc(1)
inc(2)
inc(3)
...
inc(9)
```

Every time:

```text
PUSH inc AR
execute
POP inc AR
```

Never 10 `inc` frames at once because this is **iteration, not recursion**.

That difference is exam-important.

---

# 13. Compare with recursion

Loop:

```c
while(...)
    inc();
```

behaves:

```text
main
 ├── inc
 ├── inc
 ├── inc
 └── inc
```

Each `inc` finishes before next starts.

So stack never becomes:

```text
inc
inc
inc
main
```

But recursion:

```c
fact(4)
```

does:

```text
fact(1)
fact(2)
fact(3)
fact(4)
main
```

because each call waits for the deeper call.

---

# 14. All the 17/18-series CT questions are basically this same question

Your CT collection repeatedly uses tiny programs such as:

```c
c = inc(x);
```

or:

```c
x = dec(j);
```

or:

```c
i = incr(i);
```

and then asks:

* activation tree
* activation record/stack growth
* assembly
* assembly **using stack allocation + AR**

For example, the 17-series CT pages repeatedly use this format. 

The function name is irrelevant.

### Every call:

```c
answer = func(argument);
```

turns into the exact same mental block:

```text
get argument
      ↓
move SP to new frame
      ↓
put argument in frame
      ↓
save return address
      ↓
BR function
      ↓
function computes answer
      ↓
save returned value
      ↓
BR *0(SP)
      ↓
read answer
      ↓
restore SP
      ↓
store answer
```

Learn this picture, not 12 separate answers.

---

# 15. `if` question

Suppose CT gives:

```c
if(x > 0)
    c = inc(x);
else
    x++;
```

Do not mix the two concepts.

First:

```text
CONTROL FLOW
```

decides true/false.

Then only true branch contains the function-call skeleton.

Conceptually:

```text
LD R1,x(SP)

if x <= 0 → ELSE

; TRUE:
LD R1,x(SP)

ADD SP,SP,#main.recordSize
ST ARG(SP),R1
ST 0(SP),#BACK
BR INC

BACK:
LD R1,RET(SP)
SUB SP,SP,#main.recordSize
ST c(SP),R1
BR DONE

ELSE:
LD R1,x(SP)
ADD R1,R1,#1
ST x(SP),R1

DONE:
...
```

Don't memorize this whole code.

Remember:

```text
IF part = normal code generation

FUNCTION part = AR call skeleton
```

---

# 16. `for` and `while` questions

Same idea.

For:

```c
for(j=0; j<1; j++)
    x = dec(j);
```

mentally rewrite:

```c
j = 0;

while(j < 1) {
    x = dec(j);
    j = j+1;
}
```

Then:

```text
initialize
   ↓
test
   ↓
call dec using AR
   ↓
come back
   ↓
increment
   ↓
test again
```

No new Runtime concept.

---

# 17. What goes inside the AR for these small exam snippets?

Suppose:

```c
int dec(int q) {
    return q-1;
}
```

A useful exam drawing is:

```text
AR(dec)

┌───────────────────┐
│ actual param q    │
├───────────────────┤
│ returned value    │
├───────────────────┤
│ control link      │
├───────────────────┤
│ saved status      │ ← includes return address
├───────────────────┤
│ local data        │
├───────────────────┤
│ temporaries       │
└───────────────────┘
```

If there is no nested procedure:

```text
access link may be unnecessary/unused
```

If there are no locals:

```text
local-data part may be empty
```

Theoretical AR still has the general seven-field model; a real implementation uses only what it needs. Your slides explicitly note that fields/positions may vary. 

---

# 18. One thing students often do wrong

They write:

```text
BR INC
```

and think that's enough.

No.

How would `inc` know where to return?

That's why:

```text
ST 0(SP),#BACK
```

must happen before:

```text
BR INC
```

Mental picture:

```text
ST return address
= leave a note:

"After you're finished,
come back HERE."
```

Then:

```text
BR *0(SP)
```

= read the note.

This is probably the single most important concept in the entire mixed Runtime/assembly question.

---

# 19. Another common mistake: restore SP too early

Wrong:

```text
BACK:
SUB SP,SP,#main.recordSize
LD R1,RET(SP)
```

Why wrong?

After `SUB`, SP already points back to **main**.

You lost the easy reference to the callee's return-value field.

Safer conceptual order:

```text
BACK:
LD  R1, RET(SP)
SUB SP,SP,#main.recordSize
```

First take the gift.

Then throw away the box.

---

# 20. The 2020 pass-by-reference trap

There was one other tiny gap.

The 2020 final asks about a function with parameters passed by reference and calls it with the **same variable twice**. 

Suppose:

```c
f(x,y) {
    x = x+1;
    y = y+2;
    return x+y;
}

a = 3;
f(a,a);
```

Pass by reference means:

```text
x does NOT contain a copy of a
y does NOT contain a copy of a

x → a
y → a
```

Mental picture:

```text
x ─┐
   │
   ├────→ a = 3
   │
y ─┘
```

Now execute.

### `x = x+1`

Since `x → a`:

```text
a = 4
```

Now:

```text
x sees 4
y sees 4
```

because both refer to `a`.

### `y = y+2`

```text
a = 6
```

Now:

```text
x sees 6
y sees 6
```

Return:

```text
x+y
= 6+6
= 12
```

# Answer = `12`

The trap is thinking:

```text
x = 4
y = 5
return 9
```

That would treat them like independent copies.

They aren't.

### Memory trick

```text
Pass by value:
two photocopies

Pass by reference:
two fingers pointing at the SAME box
```

---

# 21. The complete exam algorithm

For **any** question saying:

> “Write assembly using stack allocation and activation record”

use this workflow:

| What you see      | What you do                                     |
| ----------------- | ----------------------------------------------- |
| normal assignment | `LD → operation → ST`                           |
| condition         | `SUB/compare → conditional BR`                  |
| loop              | label → test → body → BR back                   |
| function call     | **create AR → save return address → BR callee** |
| parameter         | store it in callee's parameter field            |
| `return value`    | put it in returned-value field/register         |
| callee return     | `BR *0(SP)`                                     |
| back in caller    | get result → restore `SP`                       |
| recursive call    | same call sequence again → another AR           |

---

# 22. The only code block you should actually memorize

This:

```text
; ===== CALLER =====

; evaluate argument first
LD   R1, argument

; push/create callee AR
ADD  SP, SP, #caller.recordSize

; pass argument
ST   ARG(SP), R1

; remember where to return
ST   0(SP), #BACK

; call
BR   CALLEE


BACK:
; receive answer while callee AR still exists
LD   R1, RET(SP)

; pop callee AR
SUB  SP, SP, #caller.recordSize

; use answer
ST   result, R1


; ===== CALLEE =====

CALLEE:
LD   R1, ARG(SP)

; do whatever function does
...

; return result
ST   RET(SP), R1

; go back to caller
BR   *0(SP)
```

Everything else in the CT/final questions is just:

> **replace `argument`, `result`, and the function body.**

---

# 23. One-screen mental picture

```text
               CALLER
                 │
                 │ evaluate argument
                 ▼
          ┌──────────────┐
          │ ADD SP       │  ← create frame
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │ parameter    │
          │ return addr  │
          └──────┬───────┘
                 │
               BR FUNC
                 │
                 ▼
             CALLEE
                 │
             calculate
                 │
          save return value
                 │
             BR *0(SP)
                 │
                 ▼
               BACK
                 │
           get return value
                 │
             SUB SP
                 │
                 ▼
           CALLER AGAIN
```

---

# 24. Now your whole Runtime question set reduces to 8 patterns

Your collection itself shows the recurring families: activation record/tree, recursive stack growth, storage allocation, heap/reference counting, access links/displays, call-return sequence, parameter passing, and mixed stack-allocation assembly. 

So at this point the useful map is:

```text
1. Activation Record
   → know 7 fields

2. Activation Tree
   → parent = caller

3. Stack grow/shrink
   → call PUSH, return POP

4. Recursive problem
   → each live call gets separate AR

5. Static / Stack / Heap
   → comparison table

6. Access Link / Display
   → staircase vs elevator

7. Garbage / Reference Count
   → +1 / -1 / zero = garbage

8. Assembly + AR
   → ADD SP
     save return address
     BR callee
     BR *0(SP)
     SUB SP
```

That final `#8` was the missing piece.

## If you can reproduce this without looking:

```text
ADD SP,SP,#caller.recordSize
ST  *SP,#BACK
BR  CALLEE

BACK:
SUB SP,SP,#caller.recordSize

CALLEE:
...
BR *0(SP)
```

and understand **why each line exists**, then the stack-allocation/activation-record part of those 2017/18 CT + 2021/22/24 assembly questions is no longer a new problem every time. It's the **same four ideas wearing different code snippets**.
