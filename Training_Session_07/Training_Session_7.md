# Training Session 7

## Normal Forms Overview

### First Normal Form (1NF)

**Rules:**

- Each cell contains **one value**
- No arrays or lists inside a column

**Example — NOT in 1NF:**

| Student | Courses              |
|---------|----------------------|
| Alice   | Databases, Intro to AI |
| Bob     | Databases            |

**Converted to 1NF:**

| Student | Courses     |
|---------|-------------|
| Alice   | Databases   |
| Alice   | Intro to AI |
| Bob     | Databases   |

---

### Second Normal Form (2NF)

**Rules:**

- Must be in **1NF**
- No **partial dependency**

> Partial dependencies matter when we have **composite keys** (a key with multiple columns).

**Example — NOT in 2NF:**

| Student | Course      | StudentAge |
|---------|-------------|------------|
| Alice   | Databases   | 20         |
| Alice   | Intro to AI | 20         |
| Bob     | Databases   | 22         |

- **Primary Key:** (Student, Course)
- **FDs:**
  - (Student, Course) → StudentAge
  - Student → StudentAge

StudentAge **partially depends** on the key (it depends only on Student, not the full key).

> **Q:** Why is this bad?
> **A:** When Alice changes age, we need to update multiple rows.

**Solution — Split into two tables:**

| Student | StudentAge |
|---------|------------|
| Alice   | 20         |
| Bob     | 22         |

| Student | Course      |
|---------|-------------|
| Alice   | Databases   |
| Alice   | Intro to AI |
| Bob     | Databases   |

✅ This is now in **2NF**.

---

### Third Normal Form (3NF)

**Rules:**

- Must be in **2NF**
- No **transitive dependency**

**Remember transitivity:** If A → B and B → C, then A → C (by transitivity).

**Example — NOT in 3NF:**

| Student | Major | Office |
|---------|-------|--------|
| Alice   | AI    | U143   |
| Bob     | CS    | U145   |

- **Primary Key:** Student
- **FDs:** Student → Major, Major → Office
- So: Student → Office (due to transitivity!)

> **Q:** Why is this bad?
> **A:** If CS changes office, we need to update every CS student row (risk for update anomalies).

**Solution — Convert to 3NF:**

| Student | Major |
|---------|-------|
| Alice   | AI    |
| Bob     | CS    |

| Major | Office |
|-------|--------|
| AI    | U143   |
| CS    | U145   |

✅ This is now in **3NF**.

---

### Boyce-Codd Normal Form (BCNF)

**Rule:** For every dependency A → B, A must be a **superkey**.

> Remember, a superkey uniquely identifies a row.

**Example — In 3NF but NOT in BCNF:**

| Student | Course      | Teacher     |
|---------|-------------|-------------|
| Alice   | Databases   | Panagiotis  |
| Bob     | Databases   | Panagiotis  |
| Alice   | Intro to AI | Matteo      |

- **Primary Key:** (Student, Course)
- **FD:** Course → Teacher (each course only has one teacher)

**Problem:** Course is **not** a superkey, but Course → Teacher exists as a FD — this violates BCNF.

> **Q:** Why is this bad?
> **A:** If Databases gets a new teacher, we must update multiple rows.

**Solution — Split into:**

| Course      | Teacher    |
|-------------|------------|
| Databases   | Panagiotis |
| Intro to AI | Matteo     |

| Student | Course      |
|---------|-------------|
| Alice   | Databases   |
| Alice   | Intro to AI |
| Bob     | Databases   |

✅ Now every dependency has a superkey on the left side — this is in **BCNF**.

---

## Exercise 3.3.1

**BCNF definition:** A relation R is in Boyce–Codd Normal Form (BCNF) if, for every non-trivial FD X → A holding in R, X is a superkey of R. Any FD where X is not a superkey is a BCNF violation.

**BCNF decomposition algorithm:** To decompose R using a BCNF violation X → Y:
1. R<sub>1</sub> = X⁺
2. R<sub>2</sub> = R − (X⁺ − X)

Repeat on each result until no violations remain. Since X⁺ includes all attributes determined by X, multiple FDs with the same LHS(LeftHandSide) are handled automatically without needing to expand them first.

---

### a) R(A, B, C, D) with FDs: AB → C, C → D, D → A

**Step 1 — Find keys by computing closures.**

| Set | Closure | Key? |
|-----|---------|------|
| A | {A} | No |
| B | {B} | No |
| C | {A, C, D} | No |
| D | {A, D} | No |
| AB | {A, B, C, D} | **Yes** |
| BC | {A, B, C, D} | **Yes** |
| BD | {A, B, C, D} | **Yes** |

**Keys: AB, BC, BD.** Every attribute (A, B, C, D) appears in at least one key, so all attributes are prime.

**Step 2 — Identify BCNF violations (LHS not a superkey).**

| FD | LHS closure | LHS a superkey? | Violation? |
|----|-------------|-----------------|------------|
| C → D | {A, C, D} | No | **Yes** |
| D → A | {A, D} | No | **Yes** |
| C → A (derived: C → D → A) | {A, C, D} | No | **Yes** |

Note: AB → D (derived from AB → C → D) is not a violation because AB is a key.

**Step 3 — Decompose.**

Take violation **C → D**:
- X = C, X⁺ = {A, C, D} (via C → D → A)
- R<sub>1</sub> = {A, C, D}
- R<sub>2</sub> = {A, B, C, D} − ({A, C, D} − {C}) = {B, C} — no non-trivial FDs, key BC, BCNF ✓

Project FDs onto R<sub>1</sub>(A, C, D):
- C → D and D → A hold.

Key of R<sub>1</sub>: C⁺ = {A, C, D} ✓. **Key of R<sub>1</sub>: C.**

BCNF violation in R<sub>1</sub>: **D → A** (D⁺ = {A, D} ≠ {A, C, D}).

Take violation **D → A** in R<sub>1</sub>:
- X = D, X⁺ = {A, D}
- R<sub>11</sub> = {A, D} — key D (D → A), BCNF ✓
- R<sub>12</sub> = {A, C, D} − ({A, D} − {D}) = {C, D} — key C (C → D), BCNF ✓

**Final decomposition: {A, D}, {C, D}, {B, C}**

---

### b) R(A, B, C, D) with FDs: B → C, B → D

**Keys.** B⁺ = {B, C, D}; AB⁺ = {A, B, C, D}. **Key: AB.** Prime: A, B. Non-prime: C, D.

**BCNF violations.**

| FD | LHS closure | Violation? |
|----|-------------|------------|
| B → C | {B, C, D} | **Yes** |
| B → D | {B, C, D} | **Yes** |

**Decompose.** Take violation **B → C**:
- X = B, X⁺ = {B, C, D} (via B → C, B → D)
- R<sub>1</sub> = {B, C, D} — key B, BCNF ✓
- R<sub>2</sub> = {A, B, C, D} − ({B, C, D} − {B}) = {A, B} — no FDs, key AB, BCNF ✓

**Final decomposition: {B, C, D}, {A, B}**

---

### c) R(A, B, C, D) with FDs: AB → C, BC → D, CD → A, AD → B

**Keys.** AB⁺ = BC⁺ = CD⁺ = AD⁺ = {A, B, C, D}. All single attributes close only to themselves; AC⁺ = {A, C}, BD⁺ = {B, D}. **Keys: AB, BC, CD, AD.** All attributes are prime.

**BCNF violations.** Every given FD has a key as its LHS. No other non-trivial FDs with a non-superkey LHS exist (all single-attribute and remaining two-attribute closures are incomplete).

**No BCNF violations. R is already in BCNF.**

---

### d) R(A, B, C, D) with FDs: A → B, B → C, C → D, D → A

**Keys.** The FDs form a cycle, so A⁺ = B⁺ = C⁺ = D⁺ = {A, B, C, D}. **Keys: A, B, C, D** (every single attribute is a key).

**BCNF violations.** Every FD has a key as its LHS.

**No BCNF violations. R is already in BCNF.**

---

### e) R(A, B, C, D, E) with FDs: AB → C, DE → C, B → D

**Keys.** A and E never appear on any RHS, so every key must contain both. AB⁺ = {A, B, C, D} (missing E); ABE⁺ = {A, B, C, D, E} ✓. No smaller set containing A and E reaches all attributes. **Key: ABE.** Prime: A, B, E. Non-prime: C, D.

**BCNF violations.**

| FD | LHS closure | Violation? |
|----|-------------|------------|
| B → D | {B, D} | **Yes** |
| DE → C | {C, D, E} | **Yes** |
| AB → C | {A, B, C, D} | **Yes** |
| BE → C (derived: B → D, then DE → C) | {B, C, D, E} | **Yes** |

**Decompose.**

Take violation **B → D**:
- X = B, X⁺ = {B, D}
- R<sub>1</sub> = {B, D} — key B, BCNF ✓
- R<sub>2</sub> = {A, B, C, D, E} − ({B, D} − {B}) = {A, B, C, E}

Project FDs onto R<sub>2</sub>(A, B, C, E): AB → C and BE → C hold (B⁺ in R = {B,D}, BE⁺ in R = {B,C,D,E}, both projected to {A,B,C,E}). Key of R<sub>2</sub>: ABE (no smaller set reaches all of R12).

BCNF violation in R12: **AB → C** (AB⁺ = {A, B, C} ≠ {A, B, C, E}).

Take violation **AB → C** in R12:
- X = AB, X⁺ = {A, B, C}
- R121 = {A, B, C} — key AB, BCNF ✓
- R122 = {A, B, C, E} − ({A, B, C} − {A, B}) = {A, B, E} — no non-trivial FDs, key ABE, BCNF ✓

**Final decomposition: {B, D}, {A, B, C}, {A, B, E}**

*(Note: DE → C is not preserved in any single relation; BCNF decomposition does not guarantee dependency preservation.)*

---

### f) R(A, B, C, D, E) with FDs: AB → C, C → D, D → B, D → E

**Keys.** A never appears on any RHS, so every key contains A.
- AB⁺ = {A, B, C, D, E} ✓ → Key **AB**
- AC⁺ = {A, B, C, D, E} ✓ → Key **AC**
- AD⁺ = {A, B, C, D, E} ✓ → Key **AD**
- AE⁺ = {A, E} (E determines nothing) — not a key.

**Keys: AB, AC, AD.** Prime: A, B, C, D. Non-prime: E only.

**BCNF violations.**

| FD | LHS closure | LHS a superkey? | Violation? |
|----|-------------|-----------------|------------|
| AB → C | {A, B, C, D, E} | Yes (AB is a key) | No |
| C → D | {B, C, D, E} | No | **Yes** |
| D → B | {B, D, E} | No | **Yes** |
| D → E | {B, D, E} | No | **Yes** |
| C → B (derived: C → D → B) | {B, C, D, E} | No | **Yes** |
| C → E (derived: C → D → E) | {B, C, D, E} | No | **Yes** |

**Decompose.**

Take violation **C → D**:
- X = C, X⁺ = {B, C, D, E} (via C → D → B, D → E)
- R<sub>1</sub> = {B, C, D, E}
- R<sub>2</sub> = {A, B, C, D, E} − ({B, C, D, E} − {C}) = {A, C} — no non-trivial FDs, key AC, BCNF ✓

Project FDs onto R<sub>1</sub>(B, C, D, E): C → D, D → B, and D → E hold. Key of R<sub>1</sub>: C (C⁺ = {B, C, D, E}).

BCNF violation in R<sub>1</sub>: **D → B** (D⁺ = {B, D, E} ≠ {B, C, D, E}).

Take violation **D → B** in R<sub>1</sub>:
- X = D, X⁺ = {B, D, E}
- R<sub>11</sub> = {B, D, E} — FDs: D → B, D → E; key D, BCNF ✓
- R<sub>12</sub> = {B, C, D, E} − ({B, D, E} − {D}) = {C, D} — key C (C → D), BCNF ✓

**Final decomposition: {B, D, E}, {C, D}, {A, C}**

---

## Exercise 3.3.2

**Setup:** R(A, B, C, D) with FDs A → B and A → C. Only key: {A, D} (A⁺ = {A, B, C} ≠ all attributes). Both A → B and A → C are BCNF violations.

---

### Strategy 1: Decompose by A → B first (without expanding)

**Step 1.** Take violation A → B:
- R1 = {A, B} — key A, BCNF ✓
- R2 = {A, C, D} — project FDs: A → C holds (B is gone). Key of R2: A⁺ = {A, C}, so key is {A, D}.

**Step 2.** R2 has violation A → C (A⁺ = {A, C} ≠ {A, C, D}). Decompose:
- R21 = {A, C} — key A, BCNF ✓
- R22 = {A, D} — no FDs, key AD, BCNF ✓

**Result: {A, B}, {A, C}, {A, D}** — 3 relations.

---

### Strategy 2: Expand A → B to A → BC first, then decompose

Since A → B and A → C share the same LHS A, expand: **A → BC**.

**Step 1.** Take violation A → BC:
- R1 = {A, B, C} — FDs: A → B, A → C; key A, BCNF ✓
- R2 = {A, D} — no FDs, key AD, BCNF ✓

**Result: {A, B, C}, {A, D}** — 2 relations.

---

### Comparison

The two strategies yield **different decompositions**. Strategy 1 produces three relations; Strategy 2 produces two. Both are valid lossless-join BCNF decompositions, but they are not the same: Strategy 1 separates B and C into individual relations {A, B} and {A, C}, while Strategy 2 keeps them together in {A, B, C}.

The join of Strategy 1's {A, B} and {A, C} on A gives exactly {A, B, C} (since A is the key of each), so the two decompositions are informationally equivalent—but they differ structurally. **Expanding the violation before decomposing produces a more compact result with fewer relations.**

---

## Exercise 3.5.1

**3NF definition:** A relation R is in Third Normal Form (3NF) if, for every non-trivial FD X → A, at least one of the following holds:
1. X is a superkey of R, **or**
2. A is a prime attribute (a member of some candidate key of R).

3NF is weaker than BCNF: it permits FDs with a non-superkey LHS provided the RHS attribute is prime.

**3NF synthesis algorithm:**
1. Find a minimal basis for the FDs.
2. Combine FDs with the same LHS.
3. Create one relation per (combined) FD.
4. If no resulting relation contains a key of R, add a relation consisting of the attributes of any one key.
5. Remove any relation whose attribute set is a subset of another's.

The synthesis always produces a lossless-join, dependency-preserving decomposition.

---

### a) R(A, B, C, D) with FDs: AB → C, C → D, D → A

**Keys and prime attributes:** Keys are AB, BC, BD (from 3.3.1a). Every attribute is prime.

**3NF violations:** A violation requires a non-prime RHS. Since every attribute is prime, **no 3NF violations exist.** R is already in 3NF (though not in BCNF, as shown in 3.3.1a).

---

### b) R(A, B, C, D) with FDs: B → C, B → D

**Keys and prime attributes:** Key: AB. Prime: A, B. Non-prime: C, D.

**3NF violations:**
- B → C: B not a superkey, C not prime. **Violation.**
- B → D: B not a superkey, D not prime. **Violation.**

**Synthesis:**
1. Minimal basis: {B → C, B → D} (already minimal; equivalently written B → CD).
2. Combine: B → CD.
3. Relation: {B, C, D}.
4. Key check: {B, C, D} does not contain the key AB → add {A, B}.

**Decomposition: {B, C, D}, {A, B}**

---

### c) R(A, B, C, D) with FDs: AB → C, BC → D, CD → A, AD → B

**Keys:** AB, BC, CD, AD. All attributes prime. **No 3NF violations.** R is already in 3NF (and in BCNF).

---

### d) R(A, B, C, D) with FDs: A → B, B → C, C → D, D → A

**Keys:** A, B, C, D. All attributes prime. **No 3NF violations.** R is already in 3NF (and in BCNF).

---

### e) R(A, B, C, D, E) with FDs: AB → C, DE → C, B → D

**Keys and prime attributes:** Key: ABE. Prime: A, B, E. Non-prime: C, D.

**3NF violations:**
- B → D: B not a superkey, D not prime. **Violation.**
- DE → C: DE not a superkey, C not prime. **Violation.**
- AB → C: AB not a superkey, C not prime. **Violation.**
- BE → C (derived: B → D, DE → C): BE not a superkey, C not prime. **Violation.**

**Synthesis:**
1. Minimal basis: {AB → C, DE → C, B → D}. Removing any one FD loses its derived closure (e.g., without B → D, the attribute D cannot be derived from B).
2. No two FDs share the same LHS.
3. Relations: {A, B, C}, {C, D, E}, {B, D}.
4. Key check: none contains ABE → add {A, B, E}.

**Decomposition: {A, B, C}, {C, D, E}, {B, D}, {A, B, E}**

All four relations are in BCNF ({A,B,C} has key AB; {C,D,E} has key DE; {B,D} has key B; {A,B,E} has key ABE with no non-trivial FDs).

---

### f) R(A, B, C, D, E) with FDs: AB → C, C → D, D → B, D → E

**Keys and prime attributes:** Keys: AB, AC, AD. Prime: A, B, C, D. Non-prime: E only.

**3NF violations:**
- AB → C: AB is a key. **No violation.**
- C → D: D is prime. **No violation.**
- D → B: B is prime. **No violation.**
- D → E: D not a superkey, E not prime. **Violation.**
- C → E (derived: C → D → E): C not a superkey, E not prime. **Violation.**

**Synthesis:**
1. Minimal basis: {AB → C, C → D, D → B, D → E} (each FD is non-redundant).
2. Combine same-LHS FDs: D → B and D → E become D → BE.
3. Relations: {A, B, C}, {C, D}, {B, D, E}.
4. Key check: {A, B, C} contains the key AB ✓. No addition needed.

**Decomposition: {A, B, C}, {C, D}, {B, D, E}**

All three relations are also in BCNF: {A,B,C} has key AB; {C,D} has key C; {B,D,E} has key D (D → B, D → E).

---

## Exercise 3.5.2

**Courses(C, T, H, R, S, G)**  
FDs: C → T, HR → C, HT → R, HS → R, CS → G

### a) Keys for Courses

**Step 1 — Find attributes that never appear on any RHS.**  
RHS attributes across all FDs: T, C, R, R, G. Attributes **H** and **S** never appear on any RHS, so every key must contain both H and S.

**Step 2 — Compute {H, S}⁺.**

| Step | Current set | FDs |
|------|-------------|----------|
| Start | {H, S} | — |
| HS → R  | {H, R, S} | HS → R |
| HR → C  | {C, H, R, S} | HR → C |
| C → T  | {C, H, R, S, T} | C → T |
| CS → G  | {C, G, H, R, S, T} | CS → G |

{H, S}⁺ = {C, G, H, R, S, T} = all attributes ✓.

**Step 3 — Check minimality.** H⁺ = {H}, S⁺ = {S}. Neither alone determines all attributes.

**The only key is {H, S}.**

### b) Verify that the given FDs are their own minimal basis

A minimal basis requires: (1) single-attribute RHS — already satisfied; (2) no redundant LHS attribute; (3) no redundant FD.

**LHS minimality:**
- C → T: single-attribute LHS, trivially minimal.
- HR → C: H⁺ = {H}, R⁺ = {R}. Neither alone determines C. Minimal.
- HT → R: H⁺ = {H}, T⁺ = {T}. Neither alone determines R. Minimal.
- HS → R: H⁺ = {H}, S⁺ = {S}. Neither alone determines R. Minimal.
- CS → G: C⁺ = {C, T}, S⁺ = {S}. Neither alone determines G. Minimal.

**FD redundancy (remove each FD and check if it can be re-derived):**
- Remove C → T: C⁺ = {C}. Cannot derive T. Not redundant.
- Remove HR → C: HR⁺ = {H, R}. Cannot derive C. Not redundant.
- Remove HT → R: HT⁺ = {H, T}. Cannot derive R. Not redundant.
- Remove HS → R: HS⁺ = {H, S}. Cannot derive R. Not redundant.
- Remove CS → G: CS⁺ = {C, S, T}. Cannot derive G. Not redundant.

**The given FDs are their own minimal basis.** ✓

### c) 3NF synthesis and BCNF check

**Synthesis:**
1. Minimal basis = the given FDs (verified above).
2. No two FDs share the same LHS.
3. Create one relation per FD:

| FD | Relation |
|----|----------|
| C → T | {C, T} |
| HR → C | {H, R, C} |
| HT → R | {H, T, R} |
| HS → R | {H, S, R} |
| CS → G | {C, S, G} |

4. Key check: {H, S, R} contains the key {H, S} ✓. No addition needed.

**Decomposition: {C, T}, {H, R, C}, {H, T, R}, {H, S, R}, {C, S, G}**

**BCNF check:** In each relation, the LHS of its defining FD is the only key (C is the key of {C,T}; HR is the key of {H,R,C}; HT is the key of {H,T,R}; HS is the key of {H,S,R}; CS is the key of {C,S,G}). No other non-trivial FDs hold within any individual relation. **All relations are in BCNF.**

---

## Exercise 3.5.3

**Stocks(B, O, I, S, Q, D)**  
FDs: S → D, I → B, IS → Q, B → O

### a) Keys for Stocks

**Attributes never on any RHS:** RHS attributes are D, B, Q, O. **I** and **S** never appear on any RHS, so every key must contain both.

**Compute {I, S}⁺:**

| Step | Current set | FD  |
|------|-------------|----------|
| Start | {I, S} | — |
| I → B  | {B, I, S} | I → B |
| B → O  | {B, I, O, S} | B → O |
| IS → Q  | {B, I, O, Q, S} | IS → Q |
| S → D  | {B, D, I, O, Q, S} | S → D |

{I, S}⁺ = {B, D, I, O, Q, S} = all attributes ✓. I⁺ = {B, I, O}, S⁺ = {D, S}; neither alone is sufficient.

**The only key is {I, S}.**

### b) Verify that the given FDs are their own minimal basis

**LHS minimality:**
- S → D, I → B, B → O: single-attribute LHS, trivially minimal.
- IS → Q: I⁺ = {B, I, O}, S⁺ = {D, S}. Neither alone determines Q. Minimal.

**FD redundancy:**
- Remove S → D: S⁺ = {S}. Cannot derive D. Not redundant.
- Remove I → B: I⁺ = {I}. Cannot derive B. Not redundant.
- Remove IS → Q: IS⁺ = {B, D, I, O, S}. Cannot derive Q. Not redundant.
- Remove B → O: B⁺ = {B}. Cannot derive O. Not redundant.

**The given FDs are their own minimal basis.** ✓

### c) 3NF synthesis and BCNF check

**Synthesis:**
1. Minimal basis = the given FDs (verified above).
2. No two FDs share the same LHS.
3. Create one relation per FD:

| FD | Relation |
|----|----------|
| S → D | {S, D} |
| I → B | {I, B} |
| IS → Q | {I, S, Q} |
| B → O | {B, O} |

4. Key check: {I, S, Q} contains the key {I, S} ✓. No addition needed.

**Decomposition: {S, D}, {I, B}, {I, S, Q}, {B, O}**

**BCNF check:** Each relation's defining FD has its LHS as the only key (S is key of {S,D}; I is key of {I,B}; IS is key of {I,S,Q}; B is key of {B,O}). Within {I, S, Q}, neither I alone (I⁺ projected = {I}) nor S alone (S⁺ projected = {S}) determines Q; IS is the only key. **All relations are in BCNF.**
