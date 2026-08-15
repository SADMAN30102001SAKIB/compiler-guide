**Chapter 1: Introduction + Basic Compiler Concepts**
**Chapter 2: CFG / Derivation / Parse Tree / Ambiguity / Precedence basics**
**Chapter 3: Lexical Analysis**

* mapped them against **2011–2024 semester questions**.

The nice part: this whole section is much more repetitive than it looks. You do **not** need to study 78 + 15 + 149 pages independently.

---

# 1. What the exam actually keeps repeating

From the papers, these are the real question engines:

| Priority | Question family                               | Trend                                                    |
| -------- | --------------------------------------------- | -------------------------------------------------------- |
| 🔥🔥🔥   | **Compiler phases + output of each phase**    | Extremely repeated, including 2021–24                    |
| 🔥🔥🔥   | **Compiler vs Interpreter + Java/JIT**        | Very high, especially recent                             |
| 🔥🔥🔥   | **Token / Pattern / Lexeme + attributes**     | Core lexical question                                    |
| 🔥🔥🔥   | **Role of lexical analyzer + errors**         | Repeated many times                                      |
| 🔥🔥     | **Symbol table**                              | Old + 2024 comeback                                      |
| 🔥🔥     | **CFG → derivation → parse tree → ambiguity** | Recurring                                                |
| 🔥🔥     | **Input buffering + buffer pair + sentinel**  | Repeated older exam favorite                             |
| 🔥🔥     | **Regular expression + transition diagram**   | Heavy historically                                       |
| 🔥🔥     | **LEX / lexical-analyzer generation**         | Several papers                                           |
| 🔥       | **NFA → DFA**                                 | Very important historically, almost disappeared recently |
| 🔥       | **Cross compiler / bootstrapping**            | 2019–21 cluster                                          |
| 🔥       | **LEX + YACC extras**                         | Legacy/past-paper extra                                  |

The recent trend is obvious. **2024 alone asks compiler phases, compiler vs interpreter, JIT, and symbol table**, while 2023/2022/2021 again contain phase-output questions.  

So if time is limited:

> **Compiler phases > Compiler/Interpreter/JIT > Lexical fundamentals > CFG basics > Buffering/Regex/Transition diagram > legacy NFA-DFA/bootstrapping**

---

# 2. ENGINE 1 — Compiler, Interpreter, Java, JIT

## Compiler

Mental picture:

**Whole program → translate → target program → run later**

A compiler reads a program in a **source language** and produces an equivalent program in a **target language**. 

```text
Source Program
      ↓
   Compiler
      ↓
Target Program
      ↓
    Execute
```

## Interpreter

Mental picture:

**Read → execute → read → execute**

It directly executes operations from the source program rather than first producing a separate target program. 

### Exam comparison

| Compiler                               | Interpreter                               |
| -------------------------------------- | ----------------------------------------- |
| Produces target program                | No separate target program                |
| Translation before execution           | Executes source directly                  |
| Execution usually faster               | Execution usually slower                  |
| Errors mostly found during compilation | Better statement-by-statement diagnostics |

The slides explicitly state that compiled machine code normally executes faster, while interpreters can give better diagnostics. 

---

## So WTF is Java — compiler or interpreter?

**Both.**

```text
Java source (.java)
       ↓ compile
Bytecode (.class)
       ↓
JVM
       ↓ interpret / JIT
Machine code
```

The uploaded slide says Java first compiles to **bytecode**, which can be interpreted by a virtual machine. JIT compilers can translate bytecode into machine language immediately before execution. 

### One-line exam answer

> Java uses a hybrid approach because Java source is compiled into bytecode, while the JVM interprets or JIT-compiles that bytecode.

This directly solves the Java questions from **2018, 2021 and 2024**.

---

# 3. ENGINE 2 — Language Processing System

Memorize this one pipeline:

```text
Source Program
      ↓
 Preprocessor
      ↓
Modified Source
      ↓
   Compiler
      ↓
Assembly Code
      ↓
  Assembler
      ↓
Relocatable Machine Code
      ↓
 Linker
      ↓
Executable
      ↓
 Loader
      ↓
Memory → Execution
```

The slides specifically explain that a compiler may generate assembly because it is easier to generate and debug; the assembler then creates relocatable machine code. The linker resolves references between files and the loader brings executable code into memory.  

### Tiny definitions

**Preprocessor:** operates before compilation. Think `#include`, macros, etc.

**Compiler:** high-level source → target/assembly.

**Assembler:** assembly → machine/object code.

**Linker:** joins object files/libraries and resolves external references.

**Loader:** puts executable into memory.

This one picture answers a shocking number of **2013, 2014, 2017, 2019 and 2020** questions.

---

# 4. 🔥 ENGINE 3 — PHASES OF A COMPILER

This is the **#1 thing to master**.

Don't memorize six unrelated definitions. Imagine a sentence going through six workers:

```text
SOURCE CODE
   ↓
1. Lexical      → WORDS
   ↓
2. Syntax       → TREE
   ↓
3. Semantic     → MEANING / TYPES
   ↓
4. Intermediate Code
   ↓
5. Optimization
   ↓
6. Code Generation
   ↓
TARGET CODE
```

A compiler really does operate as a sequence of phases, transforming one representation into another. 

---

## GOD-TIER example

Suppose exam gives:

```c
X = A + B * C
```

### Phase 1 — Lexical Analysis

Break characters into **tokens**:

```text
<id,X>  =  <id,A>  +  <id,B>  *  <id,C>
```

Think:

> `"X = A + B*C"` → words

Lexical analysis is the first phase. Its output tokens can contain a token name plus an attribute pointing to a symbol-table entry. 

---

### Phase 2 — Syntax Analysis

Find grammatical structure.

Because `*` has higher precedence:

```text
        =
       / \
      X   +
         / \
        A   *
           / \
          B   C
```

Think:

> `"Which operation belongs inside which?"`

---

### Phase 3 — Semantic Analysis

Check whether the tree **makes sense**.

Examples:

* Are `A`, `B`, `C` declared?
* Are their types compatible?
* Can result be assigned to `X`?
* Is type conversion needed?

The semantic analyzer uses the syntax tree + symbol table to check semantic consistency and gather type information. 

---

### Phase 4 — Intermediate Code

```text
t1 = B * C
t2 = A + t1
X  = t2
```

This is the machine-independent middle representation.

---

### Phase 5 — Optimization

Make it better **without changing meaning**.

Example:

```text
t1 = B * C
X = A + t1
```

Maybe unnecessary temporaries/calculations can disappear.

Optimization is optional; its purpose is to improve the IR so better target code can be produced. 

---

### Phase 6 — Code Generation

Illustrative machine-style code:

```text
LOAD R0, B
MUL  R0, C
ADD  R0, A
STORE X, R0
```

The exact assembly syntax depends on the machine. The slide-level concept is: **IR → target instructions + register/memory choices**. 

---

# 5. The type-conversion variation

Exams LOVE:

```c
int x, y;
float z;

z = x + y;
```

Everything is identical until semantic analysis:

```text
x + y      → int
z          → float
```

So semantic analyzer inserts conversion:

```text
t1 = x + y
t2 = inttofloat(t1)
z  = t2
```

Done.

That's the secret behind the apparently different phase questions from **2018, 2021, 2022 and 2023**.

---

# 6. “Which phase detects this?” — FREE MARKS

Memorize only this:

| Problem                                 | Phase        |
| --------------------------------------- | ------------ |
| Invalid character / malformed token     | **Lexical**  |
| Identifier naming pattern invalid       | **Lexical**  |
| Missing `;`                             | **Syntax**   |
| Wrong bracket/statement grammar         | **Syntax**   |
| Variable not declared                   | **Semantic** |
| Wrong number/type of function arguments | **Semantic** |
| Type mismatch                           | **Semantic** |

Mental trick:

> **Letters → Lexer**
> **Grammar → Parser**
> **Meaning → Semantic**

So:

**“Function called with wrong number of arguments”** → Semantic.

**“_ cannot occur at beginning of identifier”** → Lexical.

**“Variable must be declared before use”** → Semantic.

**“Every assignment must end in ;”** → Syntax.

Exactly the kind of classification asked in the uploaded past papers.

---

# 7. Analysis vs Synthesis

Another tiny old question.

```text
             COMPILER
          /            \
     ANALYSIS          SYNTHESIS
    Front End          Back End
        ↓                  ↓
Understand source       Build target
```

Analysis breaks the source into structure and collects information in the symbol table; synthesis uses the IR + symbol-table information to construct the target program. 

**Memory:**

> Analysis = understand
> Synthesis = produce

---

# 8. Symbol Table

Think of it as the compiler's **contact list/database**.

```text
Name      Type      Scope      Location
---------------------------------------
x         int       local      ...
rate      float     global     ...
foo       function  global     ...
```

It may store:

* name
* type
* scope
* storage information
* function argument number/types
* parameter-passing method
* return type

The uploaded notes explicitly say symbol-table information is used across compiler phases.  It can also store procedure argument and return-type information. 

### Who initially creates identifier entries?

Usually the **lexical analyzer** discovers the identifier and installs/fetches its symbol-table entry.

### 2024 variation: “two suitable data structures”

This specific data-structure comparison goes beyond what these lecture slides substantively teach, but for the past-paper question:

| Linear/List table          | Hash table                             |
| -------------------------- | -------------------------------------- |
| Very simple                | Faster lookup                          |
| Search ≈ O(n)              | Average lookup ≈ O(1)                  |
| Good for tiny table        | Better for real compilers              |
| Little structural overhead | Needs hash function/collision handling |

**Best exam choice:** hash table.

---

# 9. CHAPTER 2 — you only need 5 ideas

Chapter 2 is **not another giant parser chapter**.

Its core is:

```text
CFG
 ↓
Derivation
 ↓
Parse Tree
 ↓
Ambiguity
 ↓
Associativity + Precedence
```

The slide defines a CFG using terminals, nonterminals, productions and a start symbol. 

---

## CFG = legal sentence recipe

A grammar:

```text
G = (T, N, P, S)
```

Where:

* `T` = terminals/tokens
* `N` = nonterminals
* `P` = production rules
* `S` = start symbol

Example from your slide:

```text
list  → list + digit
      | list - digit
      | digit

digit → 0 | 1 | ... | 9
```

---

# 10. Derivation

Start with `S` and repeatedly replace nonterminals.

### Leftmost derivation

Always expand **leftmost nonterminal**.

### Rightmost derivation

Always expand **rightmost nonterminal**.

Slide example for:

```text
9 - 5 + 2
```

is:

```text
list
⇒ list + digit
⇒ list - digit + digit
⇒ digit - digit + digit
⇒ 9 - digit + digit
⇒ 9 - 5 + digit
⇒ 9 - 5 + 2
```

This exact process is covered in Chapter 2. 

### Exam rule

If question says **leftmost**: always attack leftmost NT.

If says **rightmost**: always attack rightmost NT.

Nothing deeper.

---

# 11. Parse Tree

Mental picture:

> **Derivation written vertically as a family tree.**

Rules:

* root = start symbol
* internal nodes = nonterminals
* leaves = terminals/ε
* left-to-right leaves = generated string/yield



---

# 12. Parse Tree vs Syntax Tree / AST

| Parse Tree                                          | AST                                  |
| --------------------------------------------------- | ------------------------------------ |
| Shows full grammar                                  | Shows essential program structure    |
| Contains grammar helper nodes                       | Removes unnecessary nodes            |
| Usually larger                                      | Compact                              |
| Includes punctuation/operators according to grammar | Mainly meaningful operators/operands |

Example:

```text
(a + b)
```

Parse tree may contain:

```text
expr
term
factor
(
expr
+
...
)
```

AST simply:

```text
   +
  / \
 a   b
```

So the recurring statement:

> **“A parse tree contains much more information than an AST.”**

means the parse tree records **how the grammar generated the program**, whereas AST preserves mainly what is needed for later compiler phases.

This showed up again in recent papers.

---

# 13. Ambiguity

Super simple definition:

> A CFG is ambiguous if **one string can have more than one parse tree**.

Slide example:

```text
string → string + string
       | string - string
       | digit
```

For:

```text
9 - 5 + 2
```

we can interpret:

```text
(9 - 5) + 2
```

or

```text
9 - (5 + 2)
```

→ two structures → **ambiguous**. 

---

# 14. Associativity + Precedence

## Associativity = same-level operators

```text
a + b + c
```

`+` is left associative:

```text
(a+b)+c
```

Assignment is normally right associative:

```text
a = b = c
→ a = (b = c)
```

---

## Precedence = who grabs operands first?

```text
9 + 5 * 2
```

`*` has higher precedence:

```text
9 + (5*2)
```

### THE grammar to memorize

```text
expr   → expr + term
       | expr - term
       | term

term   → term * factor
       | term / factor
       | factor

factor → digit
       | ( expr )
```

That's literally the precedence construction shown in Chapter 2. 

Mental hierarchy:

```text
expr        + -
 ↓
term        * /
 ↓
factor      id / number / (...)
```

**Lower level in grammar = higher precedence.**

---

# 15. CHAPTER 3 — Lexical Analysis

The lexer is easiest if you picture a guy with a highlighter.

Input:

```c
total = price + 10;
```

Lexer highlights:

```text
total | = | price | + | 10 | ;
```

That's basically lexical analysis.

The source says its job is to read input characters, group them into **lexemes**, and output **tokens**. 

It can additionally remove whitespace/comments. 

---

# 16. 🔥 Token vs Pattern vs Lexeme

This is insanely exam-important.

Use one example:

```c
count = 25;
```

### Token = CATEGORY

```text
id
assign
number
semicolon
```

### Pattern = RULE

For identifier:

```text
letter(letter|digit)*
```

### Lexeme = ACTUAL TEXT

```text
count
=
25
;
```

So:

> **Pattern is the recipe. Lexeme is the food. Token is the food category.**

Example:

```text
Token  = id
Pattern = letter(letter|digit)*
Lexemes = x, total, price99
```

The slide defines a lexeme as the actual character sequence matching the pattern of a token. 

---

# 17. Token attributes

Why isn't just `id` enough?

Suppose:

```c
x = y + 5
```

If lexer returned:

```text
id = id + number
```

we've LOST WHICH identifier!

So it returns something like:

```text
<id, pointer-to-x>
<assign>
<id, pointer-to-y>
<plus>
<number, 5>
```

Identifiers normally use a pointer to their symbol-table entry as the attribute. 

Operators, punctuation and keywords often don't need attributes. 

---

# 18. Example: tokenize `printf`

The 2019 paper asks essentially:

```c
printf("University = %s\n", RUET);
```

Answer:

| Lexeme                | Token          |
| --------------------- | -------------- |
| `printf`              | `id`           |
| `(`                   | `(`            |
| `"University = %s\n"` | string literal |
| `,`                   | `,`            |
| `RUET`                | `id`           |
| `)`                   | `)`            |
| `;`                   | `;`            |

`printf` is not a C keyword → it is an identifier.

The lecture itself gives the same style of example with `printf("Total = %d\n", score)`. 

---

# 19. Why separate Lexical Analysis and Parsing?

Lexer:

```text
characters → tokens
```

Parser:

```text
tokens → grammatical structure
```

Why separate?

**1. Simpler design**
Parser doesn't have to care about whitespace/comments.

**2. Faster compiler**
Lexer can use special fast input-buffering techniques.

The notes explicitly identify simplification as a major reason for separation. 

Mental picture:

```text
"int   x = 5 ; // hi"
        ↓ Lexer
int id = num ;
        ↓ Parser
declaration/assignment structure
```

---

# 20. Lexical Errors

The legendary exam example:

```c
fi(a == f(x))
```

You probably meant:

```c
if(a == f(x))
```

But `fi` is a **perfectly legal identifier**.

Therefore lexer sees:

```text
<id, fi>
```

It cannot know whether you mistyped `if`.

The slides explicitly say the lexer must return `id` and another compiler phase has to deal with the problem. 

## When the lexer truly cannot continue

Possible recovery:

```text
delete extra character
insert missing character
replace incorrect character
transpose adjacent characters
```

Or **panic mode**:

> delete characters until a valid token can again be recognized.



---

# 21. 🔥 Input Buffering

Why buffer at all?

Reading **one character from disk at a time = horrible**.

Instead:

```text
BUFFER 1             BUFFER 2
[............]       [............]
       ↖ alternate loading ↗
```

Both buffers contain `N` characters. One block/system read loads many characters at once rather than one system call per character. 

Two pointers:

```text
lexemeBegin
     ↓
     totalPrice>=10
               ↑
             forward
```

**lexemeBegin** = where current lexeme starts.

**forward** = moves ahead until lexer knows where it ends. 

---

# 22. Sentinel — easiest explanation possible

Without sentinel, every character needs:

```text
1. Is this end of buffer?
2. What character is this?
```

That's unnecessary checking.

Put special `eof` at buffer end:

```text
Buffer 1                 Buffer 2
[a b c ... eof]          [x y z ... eof]
```

Now the character test itself can tell us that the buffer ended.

The slide calls the sentinel a special character that cannot be part of the source program, naturally `eof`. 

### Exam line

> Sentinels reduce the number of end-of-buffer tests and therefore speed up lexical analysis.

That's enough.

---

# 23. Regular Expressions — minimum needed for exam

Your lexical slide uses REs to specify token patterns. 

The **formal-definition / Kleene-closure questions in older papers go a little beyond what these uploaded slides explicitly teach**, so here is only the past-paper minimum.

### Core operators

```text
r | s     either r or s
rs        r followed by s
r*        zero or more r
r+        one or more r
r?        zero or one r
```

Therefore:

```text
r+ = rr*
r? = r | ε
```

### Useful patterns

Identifier:

```text
letter(letter|digit)*
```

Unsigned integer:

```text
digit+
```

Unsigned integer OR float:

```text
digit+(\.digit+)?
```

With optional exponent:

```text
digit+(\.digit+)?(E[+-]?digit+)?
```

This one pattern destroys several old semester questions at once.

---

# 24. Transition Diagram

Regular-expression pattern → **state machine picture**.

Three visual rules from the slides:

```text
→ q0       start state

((q3))     accepting state

((q4))*    accept, but retract forward pointer
```

The slide defines double circles as accepting states and `*` as requiring a one-character retract. 

---

## Identifier transition diagram

```text
START
  |
letter
  ↓
 q1 ──letter/digit──↺
  |
other
  ↓
FINAL*
```

Meaning:

1. First character must be letter.
2. Keep consuming letter/digit.
3. First non-letter/digit means identifier ended.
4. Retract it because that last character belongs to next token.

This exact behavior is described in the lexical slide. 

---

# 25. C comment transition diagram — solves 2022

Token:

```c
/* anything */
```

Use:

```text
q0 -- / --> q1
q1 -- * --> q2

q2 -- anything except * --> q2
q2 -- * --> q3

q3 -- * --> q3
q3 -- / --> ACCEPT
q3 -- anything else --> q2
```

So lexer only accepts after:

```text
*/
```

This is exactly the sort of transition-diagram construction asked in the 2022 paper. 

---

# 26. Reserved word vs Identifier

Problem:

```text
if
then
else
```

All also match:

```text
letter(letter|digit)*
```

So how does lexer know `if` is keyword?

Two approaches covered in the slide:

**Method 1:** initially insert reserved words into symbol table.

Then after recognizing an identifier-shaped string:

```text
lookup("if")
→ entry says TOKEN_IF
```

**Method 2:** separate transition diagrams for keywords.

The notes explicitly present both approaches.  

---

# 27. LEX — this is basically an automatic lexer factory

Instead of manually writing a lexical analyzer:

```text
Regular-expression patterns
           ↓
          LEX
           ↓
       lex.yy.c
           ↓
      C compiler
           ↓
   Lexical Analyzer
```

Lex converts patterns into a transition-diagram implementation and generates `lex.yy.c`. 

## Structure of LEX program

Memorize:

```text
Declarations
%%
Translation Rules
%%
Auxiliary Functions
```



Rule:

```text
Pattern    { Action }
```

Example:

```text
[0-9]+     { return NUM; }
```

---

# 28. Lex's MOST IMPORTANT RULE: LONGEST MATCH

Suppose multiple tokens can match.

Lex:

**Rule 1:** choose the **longest prefix**.

**Rule 2:** if equal length, choose the rule appearing **first** in the Lex program.



---

## Solve the weird 2020 question

Patterns:

```text
T1: a?b(b|c)*a
T2: b?(a|c)*b
T3: c?(b|a)*c
```

Input:

```text
bbaacabc
```

From beginning:

```text
T1 → bba      length 3
T2 → bb       length 2
T3 → bbaac    length 5
```

Longest = `T3`.

Remaining:

```text
abc
```

Matches:

```text
T2 → ab       length 2
T3 → abc      length 3
```

Again longest = `T3`.

### Answer

```text
T3, T3
```

That's all that monstrous-looking question is testing: **longest prefix**. The 2020 scan contains this exact token-selection problem. 

---

# 29. ⚠️ Past-paper extras NOT substantively taught in these 3 slide PDFs

This matters.

The old papers repeatedly contain:

* formal **NFA → DFA**
* ε-closure / subset construction
* cross compiler / bootstrapping
* some YACC functions

But I could not find substantive teaching of these topics in these three uploaded lecture PDFs. Chapter 3 mainly goes through lexical analysis, buffering, transition diagrams and Lex.

So **don't let these legacy questions make you study the entire automata/compiler-construction universe**.

Just learn this emergency kit.

---

# 30. NFA → DFA emergency kit

One sentence:

> **One DFA state represents a SET of NFA states.**

Algorithm:

```text
1. DFA start = ε-closure(NFA start)
2. For each DFA state T and input a:
      U = ε-closure(move(T,a))
3. Every new U becomes a DFA state.
4. DFA state is final if it contains any NFA final state.
5. Repeat until no new sets appear.
```

### Classic `(a|b)*abb`

Several old questions effectively reduce to this automaton.

Minimal DFA:

| State | a | b | Final? |
| ----- | - | - | ------ |
| A     | B | A | No     |
| B     | B | C | No     |
| C     | B | D | No     |
| D     | B | A | ✅      |

Mental meaning:

```text
A = nothing useful matched
B = suffix "a"
C = suffix "ab"
D = suffix "abb" ✅
```

The older papers repeatedly request conversion + transition table/diagram, e.g. 2011, 2014, 2017 and 2019.  

**Trend:** this was historically important, but **2021–2024 shifted heavily away from it**.

---

# 31. Cross compiler / bootstrapping emergency kit

### Cross compiler

Runs on **machine X** but produces code for **machine Y**.

```text
Runs on X
   ↓
Compiler
   ↓
Produces Y code
```

### Bootstrapping

Use an existing compiler to create a new compiler, eventually letting the compiler **compile itself**.

For the 2019 style scenario:

```text
M1 has P compiler
P compiler can target M2/Y
```

To port P:

```text
P compiler source
      ↓
cross compiler on M1
      ↓
P compiler executable for M2
```

To make P self-compiling:

```text
Run P compiler on M2
      ↓
compile P compiler's own P source
      ↓
self-hosted P compiler
```

To implement Q:

```text
write Q compiler in P
      ↓
compile using P compiler
      ↓
Q compiler for M2
```

That's basically all the 2019–21 variations are reshuffling.

---

# 32. LEX + YACC legacy functions

Only memorize:

```text
yylex()   → lexical analyzer; returns next token

yyparse() → parser generated by YACC; asks yylex() for tokens

yywrap()  → called at end of lexer input;
            usually return 1 = no more input
```

Relationship:

```text
Input
 ↓
yylex()
 ↓ tokens
yyparse()
 ↓
Parsing
```

YACC itself isn't substantively covered in these uploaded three chapter PDFs, so **don't over-study it for this set**.

---

# 33. COMPLETE 2011–2024 QUESTION MAP

Same questions with different variables are grouped under the engines above, so you're not memorizing 50 answers independently.

| Year     | Relevant questions from these topics                                                                                                                                                         | What solves them                     |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| **2024** | phases for `X=A+B*C`; compiler vs interpreter; JIT; symbol table + structures; parse tree vs AST                                                                                             | Engines 1–4, 8, 12                   |
| **2023** | compiler vs interpreter + phase outputs; another complete phase-output problem; identifier CFG/DFA                                                                                           | Engines 1, 3, legacy DFA             |
| **2022** | role + phase output; another phases problem; C-comment transition diagram; identify error phase; unambiguous grammar/derivation                                                              | Engines 3, 6, 9–14, 25               |
| **2021** | Java compiler/interpreter; compiler phases; phase responsibility; token example; self-compiling compiler; parse tree vs AST                                                                  | Engines 1–6, 16–17, legacy bootstrap |
| **2020** | linker/loader; new-machine compiler; phase output; `fi` lexical problem; closures/char classes; lexemes+values; RE/regular definition; buffering; NFA→DFA; LEX/YACC functions; longest match | Almost entire guide                  |
| **2019** | HLL→target machine; symbol table; cross compiler; token/pattern/lexeme; RE; LEX/YACC; NFA→DFA; syntax tree vs parse tree                                                                     | Engines 2–4, 8, 12, 16, 23–31        |
| **2018** | compiler/interpreter/Java; first four phase outputs; determine phase for language rules; buffering                                                                                           | Engines 1, 3, 6, 21–22               |
| **2017** | language-processing system; why assembly target; HLL→HLL translation; buffer issue; unsigned number RE; NFA→DFA; `if` vs identifier                                                          | Engines 2, 21–30                     |
| **2016** | Lex; role of lexer + separation; token attributes; sentinels; lexical errors; regex shorthand; finite automata relation                                                                      | Engines 15–30                        |
| **2015** | compiler/interpreter; compiler phases; assembly target; token/lexeme; symbol table; CFG; lexical error recovery; unsigned-number transition diagram                                          | Engines 1–26                         |
| **2014** | language-processing system; compiler/interpreter/assembler; phases; token vs terminal; parse tree; symbol table; lexer role; regex; NFA→DFA                                                  | Engines 1–30                         |
| **2013** | analysis/synthesis; language processing; compiler cousins; phase output; regex languages; identifier/unsigned-number RE; lexer/parser separation; token/pattern/lexeme                       | Engines 2–18, 23                     |
| **2012** | compiler phases; linear vs hierarchical analysis; symbol table; preprocessor; IR need                                                                                                        | Engines 2–8                          |
| **2011** | analysis/synthesis; CFG components; lexer↔parser interaction; NFA/DFA                                                                                                                        | Engines 7, 9–12, legacy DFA          |

You can see how heavily the same concepts recycle through the older papers.  

---

# 34. What changed over time?

### Older papers: 2011–2020

They liked **construction/mechanical lexical questions**:

```text
Regular Expressions
       ↓
NFA
       ↓
DFA
       ↓
Transition Diagram
       ↓
LEX
```

Plus buffering, sentinels, regex notation, etc.

### Newer papers: 2021–2024

They shifted toward **understanding the compiler as a system**:

```text
compiler phases
compiler vs interpreter
JIT
type/phase responsibility
symbol table
CFG/tree concepts
```

2024 is especially clear: Q1 alone gives **10 marks** to phases + compiler/interpreter + JIT. 

So I would **not** spend equal time on old NFA conversion and compiler phases.

---

# 35. FINAL 2-MINUTE MEMORY SHEET

```text
COMPILER
source → target program

INTERPRETER
source → execute directly

JAVA
source → bytecode → JVM → interpret/JIT


COMPILER PHASES
Lexer → Parser → Semantic → IR → Optimize → Codegen

Lexer    = characters → tokens
Parser   = tokens → tree
Semantic = type/meaning check
IR       = machine-independent middle code
Optimize = improve code
Codegen  = target code


ERROR PHASE
character/token → lexical
grammar         → syntax
meaning/type    → semantic


TOKEN / PATTERN / LEXEME
token   = category
pattern = rule
lexeme  = actual text


CFG
G = (T,N,P,S)

DERIVATION
leftmost  → replace leftmost NT
rightmost → replace rightmost NT

AMBIGUOUS
same string → 2+ parse trees


PRECEDENCE
expr   → + -
term   → * /
factor → id, number, (...)


BUFFERING
2 buffers + lexemeBegin + forward

SENTINEL
eof at buffer end → fewer checks


REGEX
| = OR
* = 0 or more
+ = 1 or more
? = optional


LEX
declarations
%%
rules
%%
functions

LEX CHOICE
1. longest match
2. tie → first rule


NFA→DFA
one DFA state = set of NFA states
```

## If your exam is extremely close

Study in exactly this order:

**① Compiler phases until you can do any expression blindly → ② Compiler vs Interpreter + Java/JIT + language-processing diagram → ③ Token/Pattern/Lexeme + attributes + errors → ④ CFG/derivation/parse tree/ambiguity/precedence → ⑤ Symbol table → ⑥ Buffer pair/sentinel → ⑦ Regex + transition diagrams + Lex → ⑧ only then NFA→DFA + bootstrapping/YACC.**

That gives the best return for the **2011–2024 pattern while still covering every recurring question family represented by these files**.
