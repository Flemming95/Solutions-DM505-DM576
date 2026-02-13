# Training Session 3 - SQL Queries

## Exercise 6.1.1: SELECT Clause Ambiguity

**Question:** If a query has a SELECT clause: `SELECT A B` how do we know whether A and B are two different attributes, or B is an alias of A?

**Answer:**

In SQL, the SELECT clause syntax determines whether we have two attributes or an alias:

1. **Two different attributes:** `SELECT A, B`
   - Attributes are separated by a comma
   - This retrieves both attribute A and attribute B

2. **B is an alias of A:** `SELECT A AS B` or `SELECT A B`
   - No comma between A and B
   - The keyword `AS` is optional in many SQL dialects
   - This retrieves attribute A and renames it to B in the result

**Example:**
```sql
-- Two different attributes:
SELECT title, year FROM Movies;

-- Alias (both are equivalent):
SELECT title AS movie_name FROM Movies;
SELECT title movie_name FROM Movies;
```

The key difference is the **comma**: its presence indicates separate attributes, while its absence (with or without AS) indicates an alias.

---

## Exercise 6.1.2: Movie Database Queries

**Schema:**
```
Movies(title, year, length, genre, studioName, producerC#)
StarsIn(movieTitle, movieYear, starName)
MovieStar(name, address, gender, birthdate)
MovieExec(name, address, cert#, netWorth)
Studio(name, address, presC#)
```

### a) Retrieve the address of the studio named MGM

```sql
SELECT address
FROM Studio
WHERE name = 'MGM';
```

### b) Retrieve the birthdate of Sandra Bullock

```sql
SELECT birthdate
FROM MovieStar
WHERE name = 'Sandra Bullock';
```

### c) Find all stars who appeared in any movie released in 1980, or any movie with "Love" in its title

```sql
SELECT DISTINCT starName
FROM StarsIn
WHERE movieYear = 1980
   OR movieTitle LIKE '%Love%';
```
**I use DISTINCT to remove potential duplicates**

### d) Find all executives whose net worth is at least 10 million dollars

```sql
SELECT name
FROM MovieExec
WHERE netWorth >= 10000000;
```

### e) Find all stars who are either male or have "Malibu" somewhere in their address

```sql
SELECT name
FROM MovieStar
WHERE gender = 'M'
   OR address LIKE '%Malibu%';
```

---

## Exercise 6.1.3: Product Database Queries

**Schema:**
```
Product(maker, model, type)
PC(model, speed, ram, hd, price)
Laptop(model, speed, ram, hd, screen, price)
Printer(model, color, type, price)
```

### a) Model number, speed, and hard-disk size of PCs priced under 1000

```sql
SELECT model, speed, hd
FROM PC
WHERE price < 1000;
```

### b) Same as (a), but rename speed → gigahertz, hd → gigabytes

```sql
SELECT model, speed AS gigahertz, hd AS gigabytes
FROM PC
WHERE price < 1000;
```

### c) Manufacturers of printers

```sql
SELECT DISTINCT maker
FROM Product
WHERE type = 'printer';
```
**I use DISTINCT to remove duplicates**

### d) Model number, memory size, and screen size of laptops costing more than 1500

```sql
SELECT model, ram, screen
FROM Laptop
WHERE price > 1500;
```

### e) All tuples in Printer for color printers (color is boolean)

```sql
SELECT *
FROM Printer
WHERE color = TRUE;
```

### f) Model number and hard-disk size for PCs with speed 3.2 and price under 2000

```sql
SELECT model, hd
FROM PC
WHERE speed = 3.2
  AND price < 2000;
```

---

## Exercise 6.1.5: Three-Valued Logic

**Question:** Given integer attributes a and b, which may be NULL, describe all (a, b) tuples that satisfy each condition using SQL's three-valued logic (TRUE, FALSE, UNKNOWN). **Remember: Comparison with NULL is always UNKNOWN.**

### a) a = 10 OR b = 20

**Note:** *We We follow the notation from the book (p. 253 - 255). Evaluation of
NULL would give the truth-value UNKNOWN.*

**Tuples that satisfy this condition:**
- (10, any) - TRUE *OR* TRUE/FALSE = **TRUE**
- (10, NULL) - TRUE *OR* UNKNOWN = **TRUE**
- (any, 20) - TRUE/FALSE *OR* TRUE = **TRUE**
- (NULL, 20) - UNKNOWN *OR* TRUE = **TRUE**

**Tuples that do NOT satisfy:**
- (any value ≠ 10, any value ≠ 20) where neither is NULL - FALSE OR FALSE = FALSE
- (UNKNOWN, UNKNOWN) - UNKNOWN OR UNKNOWN = UNKNOWN (treated as FALSE)
- (UNKNOWN, any value ≠ 20) - UNKNOWN OR FALSE = UNKNOWN (treated as FALSE)
- (any value ≠ 10, UNKNOWN) - FALSE OR UNKNOWN = UNKNOWN (treated as FALSE)

**Summary:** The condition is satisfied when at least one of these is true:
- a = 10 (regardless of b)
- b = 20 (regardless of a)

### b) a = 10 AND b = 20

**Tuples that satisfy this condition:**
- (10, 20) - TRUE AND TRUE = TRUE

**Tuples that do NOT satisfy:**
- (10, any value ≠ 20) - TRUE AND FALSE = FALSE
- (10, NULL) - TRUE AND UNKNOWN = UNKNOWN (treated as FALSE)
- (any value ≠ 10, 20) - FALSE AND TRUE = FALSE
- (NULL, 20) - UNKNOWN AND TRUE = UNKNOWN (treated as FALSE)
- (NULL, NULL) - UNKNOWN AND UNKNOWN = UNKNOWN (treated as FALSE)
- All other combinations evaluate to FALSE or UNKNOWN

**Summary:** Only (10, 20) satisfies this condition.

### c) a < 10 OR a >= 10

**Tuples that satisfy this condition:**
- (any non-NULL value, any value) - Either TRUE OR FALSE = TRUE or FALSE OR TRUE = TRUE

**Tuples that do NOT satisfy:**
- (NULL, any value) - UNKNOWN OR UNKNOWN = UNKNOWN (treated as FALSE)

**Summary:** All tuples where a is not NULL satisfy this condition. This demonstrates that in three-valued logic, the law of excluded middle does not hold when NULL values are involved.

### d) a = b

**Tuples that satisfy this condition:**
- (x, x) where x is any non-NULL value - TRUE

**Tuples that do NOT satisfy:**
- (x, y) where x ≠ y and both are non-NULL - FALSE
- (NULL, NULL) - UNKNOWN (treated as FALSE)
- (NULL, any non-NULL value) - UNKNOWN (treated as FALSE)
- (any non-NULL value, NULL) - UNKNOWN (treated as FALSE)

**Summary:** Only tuples where both a and b have the same non-NULL value satisfy this condition. Note that (NULL, NULL) does NOT satisfy the condition because NULL = NULL evaluates to UNKNOWN.

### e) a <= b

**Tuples that satisfy this condition:**
- (x, y) where x ≤ y and both are non-NULL - TRUE

**Tuples that do NOT satisfy:**
- (x, y) where x > y and both are non-NULL - FALSE
- (NULL, any value) - UNKNOWN (treated as FALSE)
- (any value, NULL) - UNKNOWN (treated as FALSE)
- (NULL, NULL) - UNKNOWN (treated as FALSE)

**Summary:** Only tuples where both a and b are non-NULL and a ≤ b satisfy this condition.

---

## Exercise 6.2.1: Movie Database with Joins

**Schema:**
```
Movies(title, year, length, genre, studioName, producerC#)
StarsIn(movieTitle, movieYear, starName)
MovieStar(name, address, gender, birthdate)
MovieExec(name, address, cert#, netWorth)
Studio(name, address, presC#)
```

### a) Who were the male stars in Titanic?

```sql
SELECT name
FROM MovieStar, StarsIn
WHERE name = starname
AND movietitle = 'Titanic'
AND gender = 'm';
```

### b) Which stars appeared in movies produced by MGM in 1995?

```sql
SELECT starname
FROM Movies
JOIN StarsIn ON title = movietitle
WHERE studioname = 'MGM' AND year = 1995;
```

### c) Who is the president of MGM studios?

```sql
SELECT MovieExec.name
FROM MovieExec, studio
WHERE cert = presc
```

### d) Which movies are longer than Gone With The Wind?

```sql
SELECT mv2.title
FROM movies mv1, movies mv2
WHERE mv1.title = 'Gone with the wind'
AND mv1.length < mv2.length;
```

Alternative approach:
```sql
SELECT title
FROM Movies
WHERE length > (
    SELECT length
    FROM Movies
    WHERE title = 'Gone with the wind'
);
```

### e) Which executives are worth more than Merv Griffin?

```sql
SELECT ME1.name
FROM MovieExec ME1
JOIN MovieExec ME2 ON ME2.name = 'Merv Griffin'
WHERE ME1.netWorth > ME2.netWorth;
```

Alternative approach:
```sql
SELECT name
FROM MovieExec
WHERE netWorth > (
    SELECT netWorth
    FROM MovieExec
    WHERE name = 'Merv Griffin'
);
```
**Spoiler: No executives with higher networth than Merv Griffin.**

---

## Exercise 6.2.3: Ships Database Queries

**Schema:**
```
Classes(class, type, country, numGuns, bore, displacement)
Ships(name, class, launched)
Battles(name, date)
Outcomes(ship, battle, result)
```

### a) Find the ships heavier than 35000 tons

```sql
SELECT S.name
FROM Ships S
JOIN Classes C ON S.class = C.class
WHERE C.displacement > 35000;
```

### b) List the name, displacement and number of guns of the ships engaged in the battle of Guadalcanal

```sql
SELECT S.name, C.displacement, C.numGuns
FROM Ships S
JOIN Classes C ON S.class = C.class
JOIN Outcomes O ON S.name = O.ship
WHERE O.battle = 'Guadalcanal';
```

### c) List all the ships mentioned in the database

```sql
SELECT name FROM Ships
UNION
SELECT ship FROM Outcomes;
```

**Explanation:** This query uses UNION to combine ships from both tables. UNION automatically removes duplicates, so if a ship appears in both tables, it will be listed only once. Ships might appear in the Outcomes table even if they're not in the Ships table (e.g., ships that were sunk before being officially recorded), and vice versa.

### d) Find those countries that have both battleships (bs) and battlecruisers (bc)

```sql
SELECT C1.country
FROM Classes C1
WHERE C1.type = 'bs'
  AND EXISTS (
      SELECT *
      FROM Classes C2
      WHERE C2.country = C1.country
        AND C2.type = 'bc'
  );
```

Alternative approach using INTERSECT:
```sql
SELECT country
FROM Classes
WHERE type = 'bs'
INTERSECT
SELECT country
FROM Classes
WHERE type = 'bc';
```

### e) Find those ships that were damaged in one battle, but later fought in another

```sql
SELECT DISTINCT O1.ship
FROM Outcomes O1
JOIN Outcomes O2 ON O1.ship = O2.ship
JOIN Battles B1 ON O1.battle = B1.name
JOIN Battles B2 ON O2.battle = B2.name
WHERE O1.result = 'damaged'
  AND O2.battle <> O1.battle
  AND B2.date > B1.date;
```

**Explanation:** This query finds ships that:
1. Were damaged in one battle (O1)
2. Participated in a different battle (O2)
3. The second battle occurred after the first battle (date comparison)

### f) Find those battles with at least three ships of the same country

```sql
SELECT DISTINCT O.battle
FROM Outcomes O
JOIN Ships S ON O.ship = S.name
JOIN Classes C ON S.class = C.class
GROUP BY O.battle, C.country
HAVING COUNT(DISTINCT O.ship) >= 3;
```

**Explanation:** This query finds battles where at least one country had 3 or more ships participating. The DISTINCT ensures each battle is listed only once, even if multiple countries had 3+ ships in the same battle.

Alternative considering ships might be named after their class:
```sql
SELECT DISTINCT O.battle
FROM Outcomes O
JOIN (
    SELECT S.name, C.country
    FROM Ships S
    JOIN Classes C ON S.class = C.class
    UNION
    SELECT C.class AS name, C.country
    FROM Classes C
) AS ShipCountry ON O.ship = ShipCountry.name
GROUP BY O.battle, ShipCountry.country
HAVING COUNT(DISTINCT O.ship) >= 3;
```

**Note:** The second query handles the case where a ship might not be in the Ships table but is named after its class (a common naval convention).

---

