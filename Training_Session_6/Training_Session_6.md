# Training Session 6

## Exercise 3.1.1: FDs and Keys for a US People Relation

**Schema attributes:** name, SSN, street, city, state, ZIP, area code, phone (7 digits)

**Assumptions:**
- No two people share the same SSN (SSN is government-issued and unique per person).
- Two people may share the same address (e.g., family members, roommates).
- Two people may share the same phone number (e.g., a household landline).
- ZIP codes do not straddle state boundaries; each ZIP belongs to exactly one city and state.
- Area codes are contained within a single state (no cross-state area codes).
- A specific street + city + state combination maps to exactly one ZIP code.
- A phone number (area code + 7-digit number) uniquely identifies a subscriber line and thereby a person.

**Expected FDs:**

| FD | Justification |
|----|---------------|
| SSN → name, street, city, state, ZIP, area code, phone | SSN uniquely identifies one person and all their attributes |
| ZIP → city, state | Each ZIP code belongs to exactly one city and state |
| area code → state | Area codes are confined to a single state |
| street, city, state → ZIP | A specific address has exactly one ZIP code |
| area code, phone → SSN | A phone number uniquely identifies a person |

**Keys:** <br> Recall: <br>
*Superkey: a set of attributes that uniquely identifies a row.  <br>
Candidate key: A minimal superkey, meaning there are no 'unnessecary' attributes  <br>
Primary key: The candidate key that is chosen to uniquely identify the database  <br>*

- **{SSN}** — SSN alone determines all other attributes.
- **{area code, phone}** — Together they determine SSN (via the last FD above), and SSN determines everything else.

---

## Exercise 3.2.1: FDs, Keys, and Superkeys for R(A, B, C, D)

**Given FDs:** AB → C, C → D, D → A

### a) All nontrivial FDs with a single attribute on the right

First, compute attribute-set closures to find all implied FDs:

| Left-hand side | Closure | Nontrivial FDs |
|---|---|---|
| C | {A, C, D} | C → A, C → D |
| D | {A, D} | D → A |
| AB | {A, B, C, D} | AB → C, AB → D |
| AC | {A, C, D} | AC → D |
| BC | {A, B, C, D} | BC → A, BC → D |
| BD | {A, B, C, D} | BD → A, BD → C |
| CD | {A, C, D} | CD → A |
| ABC | {A, B, C, D} | ABC → D |
| ABD | {A, B, C, D} | ABD → C |
| BCD | {A, B, C, D} | BCD → A |

Single-attribute sets A and B, and the set AD and ACD, each close to only themselves or {A,D}, yielding no nontrivial FDs.

Key derived FDs (non-obvious):
- C → A (from C → D → A)
- AB → D (from AB → C → D)

### b) All keys of R

A key must have a closure equal to {A, B, C, D} and be minimal.

- AB⁺ = {A,B,C,D} ✓; neither A⁺ nor B⁺ equals {A,B,C,D} → **AB is a key**
- BC⁺ = {A,B,C,D} ✓; neither B⁺ nor C⁺ equals {A,B,C,D} → **BC is a key**
- BD⁺ = {A,B,C,D} ✓; neither B⁺ nor D⁺ equals {A,B,C,D} → **BD is a key**

No single attribute is a key. No other two-attribute set is a key (AC⁺ = {A,C,D}, AD⁺ = {A,D}, CD⁺ = {A,C,D}).

**Keys: AB, BC, BD**

### c) All superkeys that are not keys

Any proper superset of a key is a superkey but not a key, provided it still yields the full closure. The set ACD⁺ = {A,C,D} ≠ {A,B,C,D}, so ACD is not even a superkey.

**Superkeys that are not keys: ABC, ABD, BCD, ABCD**

---

## Exercise 3.2.2: FDs, Keys, and Superkeys for Three Schemas

### i) S(A, B, C, D) with FDs: A → B, B → C, B → D

**Closures of relevant subsets:**

| Set | Closure |
|-----|---------|
| A | {A, B, C, D} |
| B | {B, C, D} |
| C | {C} |
| D | {D} |
| BCD | {B, C, D} |

**a) All nontrivial FDs (single RHS(Right Hand Side)):**

- A → B (given), A → C (A→B→C), A → D (A→B→D)
- B → C (given), B → D (given)
- Any set containing A yields all four attributes; sets containing only B,C,D yield at most {B,C,D}.

**b) Keys:**

A⁺ = {A,B,C,D}, and no proper subset of {A} has full closure.

**Key: A** (the only key)

**c) Superkeys that are not keys: AB, AC, AD, ABC, ABD, ACD, ABCD**

Note: BCD is *not* a superkey since BCD⁺ = {B,C,D}.

---

### ii) T(A, B, C, D) with FDs: AB → C, BC → D, CD → A, AD → B

This schema has a symmetric, cyclic structure. Closures:

| Set | Closure |
|-----|---------|
| AB | {A,B,C,D} |
| AC | {A,C} |
| AD | {A,B,C,D} |
| BC | {A,B,C,D} |
| BD | {B,D} |
| CD | {A,B,C,D} |

**a) All nontrivial FDs (single RHS):**

- AB → C (given), AB → D (AB→C, BC→D)
- BC → D (given), BC → A (BC→D, CD→A)
- CD → A (given), CD → B (CD→A, AD→B)
- AD → B (given), AD → C (AD→B, AB→C)

**b) Keys: AB, BC, CD, AD**

Each is minimal (no single attribute has full closure, and AC⁺ = {A,C}, BD⁺ = {B,D}).

**c) Superkeys that are not keys: ABC, ABD, ACD, BCD, ABCD**

---

### iii) U(A, B, C, D) with FDs: A → B, B → C, C → D, D → A

The FDs form a cycle A → B → C → D → A, so every single attribute determines all others.

**Closures:** A⁺ = B⁺ = C⁺ = D⁺ = {A, B, C, D}

**a) All nontrivial FDs (single RHS):**

- A → B, A → C, A → D
- B → A, B → C, B → D
- C → A, C → B, C → D
- D → A, D → B, D → C

**b) Keys: A, B, C, D** (all single attributes are keys)

**c) Superkeys that are not keys:** every multi-attribute subset of {A,B,C,D} — i.e., AB, AC, AD, BC, BD, CD, ABC, ABD, ACD, BCD, ABCD.

---

## Exercise 3.2.10: Projecting FDs onto S(A, B, C)

For each part, compute X⁺ using R's FDs for every subset X ⊆ {A,B,C}, then intersect with {A,B,C} to find FDs that hold in S. A minimal basis is given.

---

### a) FDs of R: AB → DE, C → E, D → C, E → A

| X | X⁺ in R | X⁺ ∩ {A,B,C} | New FDs in S |
|---|---------|--------------|-------------|
| A | {A} | {A} | — |
| B | {B} | {B} | — |
| C | {A,C,E} | {A,C} | C → A |
| AB | {A,B,C,D,E} | {A,B,C} | AB → C |
| AC | {A,C,E} | {A,C} | — |
| BC | {A,B,C,D,E} | {A,B,C} | BC → A |

BC → A is redundant: augmenting C → A gives BC → BA, hence BC → A.

**Minimal basis: { C → A, AB → C }**

---

### b) FDs of R: A → D, BD → E, AC → E, DE → B

| X | X⁺ in R | X⁺ ∩ {A,B,C} | New FDs in S |
|---|---------|--------------|-------------|
| A | {A,D} | {A} | — |
| B | {B} | {B} | — |
| C | {C} | {C} | — |
| AB | {A,B,D,E} | {A,B} | — |
| AC | {A,B,C,D,E} | {A,B,C} | AC → B |
| BC | {B,C} | {B,C} | — |

**Minimal basis: { AC → B }**

---

### c) FDs of R: AB → D, AC → E, BC → D, D → A, E → B

| X | X⁺ in R | X⁺ ∩ {A,B,C} | New FDs in S |
|---|---------|--------------|-------------|
| A | {A} | {A} | — |
| B | {B} | {B} | — |
| C | {C} | {C} | — |
| AB | {A,B,D} | {A,B} | — |
| AC | {A,B,C,D,E} | {A,B,C} | AC → B |
| BC | {A,B,C,D,E} | {A,B,C} | BC → A |

Neither AC → B nor BC → A can be derived from the other.

**Minimal basis: { AC → B, BC → A }**

---

### d) FDs of R: A → B, B → C, C → D, D → E, E → A

| X | X⁺ in R | X⁺ ∩ {A,B,C} | New FDs in S |
|---|---------|--------------|-------------|
| A | {A,B,C,D,E} | {A,B,C} | A→B, A→C |
| B | {A,B,C,D,E} | {A,B,C} | B→A, B→C |
| C | {A,B,C,D,E} | {A,B,C} | C → A, C→B |

AC → B is redundant: augmenting C → B gives AC → AB, hence AC → B.

**Minimal basis: { A → B, B → C, C → A }**
