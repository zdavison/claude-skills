---
name: quint-language-reference
description: "Use when writing, reading, or editing Quint (.qnt) code and you need exact syntax for a language construct — types, modes (val/def/pure/action/temporal/run), modules/imports/instances, set/map/list/record/tuple/sum-type operators, action operators (x' = e, nondet, oneOf, all/any blocks), run operators (then/reps/fail/expect), or temporal operators (always/eventually/next/orKeep/mustChange/enabled/weakFair/strongFair). TRIGGER when: looking up Quint syntax, deciding between two forms of an operator, writing a match expression, declaring a sum type, structuring a non-deterministic action, building a run for tests, or recalling the dot/UFCS vs prefix call form. Mirrors quint.sh/docs/lang."
---

# Quint Language Reference

A complete syntax-and-operator reference for the Quint specification language, condensed from https://quint.sh/docs/lang. Use it as a lookup table when authoring `.qnt` files, not as a tutorial.

For any operator, there are typically **three equivalent forms**: prefix `f(x, y)`, dot/UFCS `x.f(y)`, and (for built-ins only) infix like `x + y`. All forms shown together below.

## Lexical

```quint
// identifier: [a-zA-Z_][a-zA-Z0-9_]*
"hello, world!"            // string literal

// single-line comment
some code // also a comment
/* multi-line
   comment */
```

## Types

```quint
bool int str                              // basic
IDENTIFIER_IN_CAPS                        // uninterpreted type
a b c ... z                               // type variable
Set[T]  List[T]
(T1, T2, ..., Tn)                         // tuple, n >= 2
{ name1: T1, ..., nameN: TN }             // record, n >= 1
T1 -> T2                                  // function (map)
(T1, ..., Tn) => R                        // operator, n >= 0
type T = L1(T1) | ... | Ln(Tn)            // sum type
T[T1, ..., Tn]                            // polymorphic instance
(T)                                        // parenthesized
```

### Type aliases

```quint
type Temperature = int
type Option[a] = | Some(a) | None
type MY_TYPE                              // uninterpreted, no constructors
```

## Modes (Subsumption)

A more general mode may use less general operators, but not vice versa.

| More general → Less general |
|---|
| Stateless → (none) |
| State → Stateless |
| Non-determinism → Stateless, State |
| Action → Non-determinism, Stateless, State |
| Run → Stateless, State, Action |
| Temporal → Stateless, State |

Action and Temporal are **incomparable**.

| Qualifier | Expression mode | Definition mode |
|---|---|---|
| `pure val`, `pure def` | Stateless | Stateless |
| `val`, `def` | State | State |
| `action` | Action | Action |
| `temporal` | Stateless / State / Temporal | Temporal |
| `run` | — | Run |

## Module-level constructs

```quint
module Foo {
  const N: int                            // Stateless
  const Proc: Set[str]
  assume AtLeast4 = N >= 4                // anonymous: assume _ = ...
  var name: str                           // State
  var timer: int

  pure val Nodes: Set[int] = 1.to(10)
  pure def fst(x: a, y: b): a = x
  pure def max(x: int, y: int): int = if (x > y) x else y
  pure def F(G: a => b, x: a): b = G(x)   // higher-order

  val isTimerPositive = timer >= 0
  def hasExpired(timestamp: int) = timer >= timestamp

  action init = timer' = 0
  action advance(unit: int) = timer' = timer + unit

  temporal neverNegative = always(timer >= 0)
}
```

**Recursion is forbidden.** Use `map`, `filter`, `fold`, `foldl` instead.

### Imports, exports, instances

```quint
import Math.pow                           // single name
import Math.*                             // wildcard
import Math from "./math.qnt"             // from file

export A.*                                // import doesn't re-export; export does

// Module instance with constant overrides
import Voting(Value = Set(0,1), Acceptor = Acceptor, Quorum = Quorum) as V
val MyValues = V::Value                   // qualified access via ::

// Anonymous instance — definitions injected into current namespace
import Bar(c = 0).*
```

### Namespaces

```quint
module Top {
  var x: int
  val Inner::x2 = x + x                   // :: nests namespaces
  val inv = (Inner::x2 - x == x)
}
```

### Theorems

Not supported. Use TLA+ / TLAPS for theorem proving.

## Expressions

### Literals

```quint
true   false   Bool                        // Bool = {false, true}
0   1024   100_000_000   0xAB_CD_EF       // ints (digit separators ok)
Int   Nat                                  // all ints / all naturals
"text"                                     // string
Set(...)   Map(...)   List(...)   [...]    // collection literals
```

### Two call forms (and infix for built-ins)

```quint
f(e1, ..., en)        // prefix
e1.f(e2, ..., en)     // UFCS / dot — parens required even for f()
1 + 3                 // infix (built-ins only)
```

Mnemonic prefixes for infix integer ops: `iadd isub imul idiv imod ipow iuminus ilt igt ilte igte`.

### Lambdas

```quint
x => e
(x1, ..., xn) => e
((x, y)) => e2                            // tuple unpacking
_ => e                                    // unused params
```

At least one parameter required. Lambdas may only be passed as operator arguments.

### Boolean & equality

```quint
e1 == e2          eq(e1, e2)              // equality
e1 != e2          neq(e1, e2)
not(p)            p.not()
p and q           and(p1, p2, p3, ...)    // n-ary
p or q            or(p1, p2, p3, ...)     // n-ary
p iff q           p.iff(q)
p implies q       implies(p, q)           // not(p) or q
```

### Block forms

```quint
all { p1, p2, ..., pn, }                  // Action conjunction;  actionAll(...)
any { a1, a2, ..., an, }                  // Action disjunction;  actionAny(...)
and { p1, ..., pn }                       // non-Action conjunction
or  { p1, ..., pn }                       // non-Action disjunction
```

### Conditional

```quint
if (p) e1 else e2
ite(p, e1, e2)
```

### Braces vs parentheses

```quint
{ e }    // Action mode
(e)     // non-Action only
```

## Sets

```quint
Set(e1, ..., en)
S.map(x => e)            map(S, x => e)
S.filter(x => P)         filter(S, x => P)
range(start, end)        // half-open
S.fold(init, (acc, x) => e)
S.exists(x => P)         S.forall(x => P)
e.in(S)                  S.contains(e)        in(e, S)
S.union(T)               S.intersect(T)       S.exclude(T)
S.subseteq(T)
S.powerset()             S.flatten()
S.allLists()             S.allListsUpTo(n)
S.chooseSome()           S.getOnlyElement()   // runtime-error if size != 1
S.isFinite()             S.size()
```

## Maps (functions)

```quint
Map()   Map(k1 -> v1)   Map(k1 -> v1, k2 -> v2, ...)
f.get(e)                 f.keys()
S.mapBy(x => e)                              // [x \in S |-> e]
Set((1,true),(2,false)).setToMap()
S.setOfMaps(T)                               // [S -> T]
f.set(e1, e2)                                // [f EXCEPT ![e1] = e2]
f.set(e1, e2).set(e3, e4)                    // chained
f.setBy(e1, old => old + y)                  // EXCEPT with @
f.put(k, v)                                  // (k :> v) @@ f
```

## Records

```quint
{ f1: e1, ..., fn: en }       Rec(f1, e1, ..., fn, en)
r.fld                         field(r, "fld")
r.fieldNames()
r.with("f", e)                // [r EXCEPT !.f = e]
{ f1: e1, fN: eN, ...r }      // spread / multi-field update
```

## Tuples

```quint
(e1, ..., en)                 Tup(e1, ..., en)
()                            // empty / unit
t._1, t._2, ..., t._50        item(t, idx)
tuples(S1, S2, ..., Sn)       // cartesian product
```

## Sum types

```quint
type T = L1(T1) | ... | Ln(Tn)

L_k(x)                        variant("L_k", x)

match e {
  | L_1(x_1) => e_1
  | ...
  | L_n(x_n) => e_n
}

// Example
type Elem = S(str) | I(int)
val transformed = mySet.map(e => match e {
  | S(s) => s
  | I(_) => "An int"
})
```

## Lists (sequences, 0-indexed)

```quint
[e1, ..., en]                 List(e1, ..., en)
range(start, end)             // half-open
l[i]   l.nth(i)
l.length()                    l.indices()
l.head()                      l.tail()
l.append(e)                   s.concat(t)
l.replaceAt(i, e)             l.slice(start, end)   // half-open
l.select(x => P)              // filter (SelectSeq)
l.foldl(init, (acc, v) => e)  // left fold
```

## Integers

```quint
m + n   m - n   -m   m * n   m / n   m % n   m ^ n
m < n   m > n   m <= n   m >= n
m.to(n)                       // inclusive range
Int   Nat
```

## Nested definitions

Local `val`/`def`/`nondet`/`action`/`temporal` inside another operator. Use `;` to put a definition and its body on one line.

```quint
def pow4(x) =
  val x2 = x * x
  x2 * x2

def myFunc(x) =
  def helper(a, b) = a + b
  helper(x, x)

def triple(x) =
  def add(n) = n + x; add(x, add(x, x))

action next = {
  nondet i = oneOf(0.to(10))
  all { i > n, n' = i, }
}

temporal property =
  temporal A = eventually(x > 3)
  temporal B = always(eventually(y == 0))
  A implies B
```

Inner definitions cannot exceed the outer mode's generality.

## Action operators

### Delayed assignment

```quint
x' = e        assign(x, e)        x.assign(e)
```

Each variable is assigned **at most once per step**. RHS reads the current state, not the assigned value.

```quint
all { x' = 4, x + 1 > 0 }      // x + 1 reads pre-step x
```

### Non-deterministic choice

```quint
nondet name = oneOf(expr1)     // expr1 must be a non-empty Set
expr2

// Example
action nextSquare = {
  nondet i = oneOf(Int)
  all {
    Int.exists(j => i * i == j),
    i > x,
    x' = i,
  }
}
```

| Operator | Argument mode | Result mode |
|---|---|---|
| `oneOf(S)` | Stateless / State | Non-determinism |
| `nondet x = e1; e2` | e1: Nondet, e2: Action | Action |

### Assert

```quint
assert(condition)     // Action mode; runtime error if condition is false
```

## Run operators (finite executions / tests)

```quint
A.then(B)             then(A, B)                    // sequential composition
n.reps(i => A(i))     n.reps(_ => A)                // repeat n times, i = 0..n-1
A.fail()              fail(A)                       // succeeds iff A returns false
A.expect(P)           expect(A, P)                  // run A; require P in resulting state
```

`reps` semantics: `n <= 0` is no-op; `n == 1` is `A(0)`; `n > 1` is `A(0).then((n-1).reps(i => A(1+i)))`.

```quint
run test = (x' = 0)
  .then(3.reps(_ => x' = x + 1))
  .then(assert(x == 3))

run testInit = (Init).then(Positive).then(ByThree).then(Even)
```

## Temporal operators

```quint
always(P)             P.always              // [] P
eventually(P)         P.eventually          // <> P
next(e)               e.next                // e' (Temporal mode only)
orKeep(A, x)          A.orKeep(x)           // [A]_x
mustChange(A, x)      A.mustChange(x)       // <A>_x
enabled(A)            A.enabled             // ENABLED A
weakFair(A, e)        A.weakFair(e)         // WF_e(A)
strongFair(A, e)      A.strongFair(e)       // SF_e(A)
guarantees(P, Q)      P.guarantees(Q)       // P -+-> Q
```

`unchanged` is **removed** — write `next(x) == x` instead.

Leads-to: `always(P implies eventually(Q))`.

In `enabled`, `weakFair`, `strongFair`, `orKeep`, `mustChange`: argument `A` is in Action mode; the whole expression is Temporal (or Run for `orKeep`).

## Quantifiers

Bounded — over sets:

```quint
S.exists(x => P)        S.forall(x => P)
S.chooseSome()
```

Unbounded — over the first-order universe (Stateless / State / Temporal, **not** Action):

```quint
existsConst(x => P)
forallConst(x => P)
chooseConst(x => P)
```

## Quick lookup table — TLA+ ↔ Quint

| TLA+ | Quint |
|---|---|
| `[]P` / `<>P` | `always(P)` / `eventually(P)` |
| `e'` (in temporal) | `next(e)` |
| `[A]_x` / `<A>_x` | `orKeep(A, x)` / `mustChange(A, x)` |
| `WF_e(A)` / `SF_e(A)` | `weakFair(A, e)` / `strongFair(A, e)` |
| `ENABLED A` | `enabled(A)` |
| `UNCHANGED x` | `next(x) == x` |
| `\E x \in S: P` / `\A x \in S: P` | `S.exists(x => P)` / `S.forall(x => P)` |
| `CHOOSE x \in S: TRUE` | `S.chooseSome()` |
| `S \X T` | `tuples(S, T)` |
| `SUBSET S` / `UNION S` | `S.powerset()` / `S.flatten()` |
| `Cardinality(S)` / `IsFiniteSet(S)` | `S.size()` / `S.isFinite()` |
| `[f EXCEPT ![k] = v]` | `f.set(k, v)` |
| `[f EXCEPT ![k] = @ + 1]` | `f.setBy(k, old => old + 1)` |
| `[x \in S \|-> e]` | `S.mapBy(x => e)` |
| `DOMAIN f` | `f.keys()` |
| `[r EXCEPT !.f = e]` | `r.with("f", e)` |
| `Append`, `Head`, `Tail`, `Len`, `SubSeq`, `\circ`, `SelectSeq` | `append`, `head`, `tail`, `length`, `slice`, `concat`, `select` |
| `A \cdot B` | `A.then(B)` |

## Common pitfalls

- **Lists are 0-indexed** (TLA+ sequences are 1-indexed).
- **`slice(start, end)` and `range(start, end)` are half-open** (`end` excluded).
- **No recursion** — use `fold` / `foldl` / `map` / `filter`.
- **Each variable assigned at most once per step.** Multiple `x' = ...` in one step is an error.
- **`x' = e` reads the current state on the RHS**, even if other clauses also assign `x`. Assignment is delayed to the next step.
- **`next` belongs to Temporal mode only.** In actions use `x' = e`.
- **`enabled`, fairness, `orKeep`, `mustChange`** take an Action-mode argument but produce a Temporal expression — they cannot be nested inside an action.
- **`getOnlyElement` errors at runtime** if the set's size is not 1.
- **Action mode requires `{ ... }` braces**, non-Action expressions use `( ... )`.
- **`import` does not re-export** — add an explicit `export A.*` in the importing module.
