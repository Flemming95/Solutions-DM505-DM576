# Training Session 4

## Exercise 6.3.1: Subqueries on Product Database

**Schema:**
```
Product(maker, model, type)
PC(model, speed, ram, hd, price)
Laptop(model, speed, ram, hd, screen, price)
Printer(model, color, type, price)
```

### a) Find the makers of PCs with a speed of at least 3.0

**Solution 1 - Using IN:**
```sql
SELECT DISTINCT maker
FROM Product
WHERE model IN (
    SELECT model
    FROM PC
    WHERE speed >= 3.0
) AND type = 'pc';
```

**Solution 2 - Using EXISTS:**
```sql
SELECT DISTINCT maker
FROM Product p
WHERE type = 'pc'
AND EXISTS (
    SELECT *
    FROM PC
    WHERE model = p.model
    AND speed >= 3.0
);
```

### b) Find the printers with the highest price

**Solution 1 - Using ALL:**
```sql
SELECT model
FROM Printer
WHERE price >= ALL (
    SELECT price
    FROM Printer
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT model
FROM Printer p1
WHERE NOT EXISTS (
    SELECT *
    FROM Printer p2
    WHERE p2.price > p1.price
);
```

### c) Find the laptops whose speed is slower than that of any PC

**Solution 1 - Using ANY:**
```sql
SELECT model
FROM Laptop
WHERE speed < ANY (
    SELECT speed
    FROM PC
);
```
*This finds laptops slower than at least one PC*

**Solution 2 - Using EXISTS:**
```sql
SELECT model
FROM Laptop l
WHERE EXISTS (
    SELECT *
    FROM PC
    WHERE speed > l.speed
);
```

### d) Find the model number of the item (PC, laptop, or printer) with the highest price

**Solution 1 - Using ALL with UNION:**
```sql
SELECT model
FROM (
    SELECT model, price FROM PC
    UNION
    SELECT model, price FROM Laptop
    UNION
    SELECT model, price FROM Printer
) AS AllProducts
WHERE price >= ALL (
    SELECT price FROM PC
    UNION
    SELECT price FROM Laptop
    UNION
    SELECT price FROM Printer
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT model
FROM (
    SELECT model, price FROM PC
    UNION
    SELECT model, price FROM Laptop
    UNION
    SELECT model, price FROM Printer
) AS AllProducts ap1
WHERE NOT EXISTS (
    SELECT *
    FROM (
        SELECT model, price FROM PC
        UNION
        SELECT model, price FROM Laptop
        UNION
        SELECT model, price FROM Printer
    ) AS AllProducts2
    WHERE AllProducts2.price > ap1.price
);
```

### e) Find the maker of the color printer with the lowest price

**Solution 1 - Using ALL:**
```sql
SELECT DISTINCT maker
FROM Product
WHERE model IN (
    SELECT model
    FROM Printer
    WHERE color = TRUE
    AND price <= ALL (
        SELECT price
        FROM Printer
        WHERE color = TRUE
    )
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT DISTINCT maker
FROM Product p
WHERE type = 'printer'
AND EXISTS (
    SELECT *
    FROM Printer pr1
    WHERE pr1.model = p.model
    AND pr1.color = TRUE
    AND NOT EXISTS (
        SELECT *
        FROM Printer pr2
        WHERE pr2.color = TRUE
        AND pr2.price < pr1.price
    )
);
```

### f) Find the maker(s) of the PC(s) with the fastest processor among all those PCs that have the smallest amount of RAM

**Solution 1 - Using ALL with nested subquery:**
```sql
SELECT DISTINCT maker
FROM Product
WHERE model IN (
    SELECT model
    FROM PC
    WHERE ram <= ALL (SELECT ram FROM PC)
    AND speed >= ALL (
        SELECT speed
        FROM PC
        WHERE ram <= ALL (SELECT ram FROM PC)
    )
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT DISTINCT maker
FROM Product p
WHERE type = 'pc'
AND EXISTS (
    SELECT *
    FROM PC pc1
    WHERE pc1.model = p.model
    AND NOT EXISTS (
        SELECT *
        FROM PC pc2
        WHERE pc2.ram < pc1.ram
    )
    AND NOT EXISTS (
        SELECT *
        FROM PC pc3
        WHERE NOT EXISTS (
            SELECT *
            FROM PC pc4
            WHERE pc4.ram < pc3.ram
        )
        AND pc3.speed > pc1.speed
    )
);
```

---

## Exercise 6.3.2: Subqueries on Ships Database

**Schema:**
```
Classes(class, type, country, numGuns, bore, displacement)
Ships(name, class, launched)
Battles(name, date)
Outcomes(ship, battle, result)
```

### a) Find the countries whose ships had the largest number of guns

**Solution 1 - Using ALL:**
```sql
SELECT DISTINCT country
FROM Classes
WHERE numGuns >= ALL (
    SELECT numGuns
    FROM Classes
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT DISTINCT country
FROM Classes c1
WHERE NOT EXISTS (
    SELECT *
    FROM Classes c2
    WHERE c2.numGuns > c1.numGuns
);
```

### b) Find the classes of ships, at least one of which was sunk in a battle

**Solution 1 - Using IN:**
```sql
SELECT DISTINCT class
FROM Ships
WHERE name IN (
    SELECT ship
    FROM Outcomes
    WHERE result = 'sunk'
);
```
*Handles ships in Ships table*

**Solution 2 - Using EXISTS:**
```sql
SELECT DISTINCT s.class
FROM Ships s
WHERE EXISTS (
    SELECT *
    FROM Outcomes o
    WHERE o.ship = s.name
    AND o.result = 'sunk'
);
```

**Alternative considering ships named after their class:**
```sql
SELECT DISTINCT class
FROM Classes
WHERE class IN (
    SELECT ship
    FROM Outcomes
    WHERE result = 'sunk'
)
OR EXISTS (
    SELECT *
    FROM Ships s, Outcomes o
    WHERE s.name = o.ship
    AND s.class = Classes.class
    AND o.result = 'sunk'
);
```

### c) Find the names of the ships with a 16 inch bore

**Solution 1 - Using IN:**
```sql
SELECT name
FROM Ships
WHERE class IN (
    SELECT class
    FROM Classes
    WHERE bore = 16
);
```

**Solution 2 - Using EXISTS:**
```sql
SELECT name
FROM Ships s
WHERE EXISTS (
    SELECT *
    FROM Classes c
    WHERE c.class = s.class
    AND c.bore = 16
);
```

### d) Find the battles in which ships of the Kongo class participated

**Solution 1 - Using IN:**
```sql
SELECT DISTINCT battle
FROM Outcomes
WHERE ship IN (
    SELECT name
    FROM Ships
    WHERE class = 'Kongo'
)
OR ship = 'Kongo';
```
*The OR handles if Kongo itself participated (ships can be named after their class)*

**Solution 2 - Using EXISTS:**
```sql
SELECT DISTINCT o.battle
FROM Outcomes o
WHERE EXISTS (
    SELECT *
    FROM Ships s
    WHERE s.name = o.ship
    AND s.class = 'Kongo'
)
OR o.ship = 'Kongo';
```

### e) Find the names of the ships whose number of guns was the largest of those ships of the same bore

**Solution 1 - Using ALL:**
```sql
SELECT DISTINCT s.name
FROM Ships s
JOIN Classes c1 ON s.class = c1.class
WHERE c1.numGuns >= ALL (
    SELECT c2.numGuns
    FROM Classes c2
    WHERE c2.bore = c1.bore
);
```

**Solution 2 - Using NOT EXISTS:**
```sql
SELECT DISTINCT s.name
FROM Ships s
JOIN Classes c1 ON s.class = c1.class
WHERE NOT EXISTS (
    SELECT *
    FROM Classes c2
    WHERE c2.bore = c1.bore
    AND c2.numGuns > c1.numGuns
);
```

---

## Exercise 5.2.1: Relational Algebra Operations

**Given relations:**
```
R(A,B): {(0, 1), (2, 3), (0, 1), (2, 4), (3, 4)}
S(B,C): {(0, 1), (2, 4), (2, 5), (3, 4), (0, 2), (3, 4)}
```

### a) πA+B,A²,B²(R)

*Projection with computed attributes: A+B, A², B²*

**Result:**
```
A+B | A² | B²
----|----|----|
1   | 0  | 1  |
5   | 4  | 9  |
6   | 4  | 16 |
7   | 9  | 16 |
```

*Note: Duplicate (1, 0, 1) from (0,1) appearing twice is removed by projection*

### c) τB,A(R)

*Sorting by B, then A in ascending order*

**Result:**
```
A | B
--|--
0 | 1
0 | 1
2 | 3
2 | 4
3 | 4
```

### e) δ(R)

*Duplicate elimination*

**Result:**
```
A | B
--|--
0 | 1
2 | 3
2 | 4
3 | 4
```

*The duplicate tuple (0, 1) is removed*

### g) γA,SUM(B)(R)

*Grouping by A and summing B values*

**Result:**
```
A | SUM(B)
--|-------
0 | 2
2 | 7
3 | 4
```

*Calculation: For A=0: 1+1=2; For A=2: 3+4=7; For A=3: 4*

### k) R ⟕ S

*Left-outer join on common attribute B*

**Finding matching tuples where R.B = S.B (all R tuples are preserved):**

S has tuples with B values: 0, 2, 2, 3, 0, 3

- From R(0,1): S has no B=1 → (0,1,NULL)
- From R(2,3): S has B=3 at (3,4) twice → (2,3,4), (2,3,4)
- From R(0,1): S has no B=1 → (0,1,NULL)
- From R(2,4): S has no B=4 → (2,4,NULL)
- From R(3,4): S has no B=4 → (3,4,NULL)

**Result:**
```
A | B | C
--|---|-----
0 | 1 | NULL
2 | 3 | 4
2 | 3 | 4
0 | 1 | NULL
2 | 4 | NULL
3 | 4 | NULL
```

*All R tuples are preserved; unmatched tuples get NULL for C*
