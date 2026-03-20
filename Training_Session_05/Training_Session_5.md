# Training Session 5

## Exercise 6.4.1: SQL Queries for Product Database (from Exercise 2.4.1)

**Schema:**
```
Product(maker, model, type)
PC(model, speed, ram, hd, price)
Laptop(model, speed, ram, hd, screen, price)
Printer(model, color, type, price)
```

### a) What PC models have a speed of at least 3.00?

```sql
SELECT DISTINCT model
FROM PC
WHERE speed >= 3.00;
```

Selects PC models filtering by speed, with DISTINCT to eliminate duplicate model numbers.

### b) Which manufacturers make laptops with a hard disk of at least 100GB?

```sql
SELECT DISTINCT maker
FROM Product
JOIN Laptop ON Product.model = Laptop.model
WHERE Laptop.hd >= 100;
```

Joins Product with Laptop to link manufacturers to laptop specs, filtering by hard disk size.

### c) Find the model number and price of all products (of any type) made by manufacturer B.

```sql
SELECT DISTINCT Product.model, price
FROM Product JOIN PC ON Product.model = PC.model
WHERE maker = 'B'
UNION
SELECT DISTINCT Product.model, price
FROM Product JOIN Laptop ON Product.model = Laptop.model
WHERE maker = 'B'
UNION
SELECT DISTINCT Product.model, price
FROM Product JOIN Printer ON Product.model = Printer.model
WHERE maker = 'B';
```

Uses UNION to combine results from all three product types; UNION inherently eliminates duplicates.

### h) Find those manufacturers of at least two different computers (PC's or laptops) with speeds of at least 2.80.

```sql
SELECT maker
FROM Product
WHERE model IN (
    SELECT model FROM PC WHERE speed >= 2.80
    UNION
    SELECT model FROM Laptop WHERE speed >= 2.80
)
GROUP BY maker
HAVING COUNT(DISTINCT model) >= 2;
```

Finds fast computers using a subquery with UNION, then groups by maker and filters those with at least two qualifying models.

---

## Exercise 6.4.2: SQL Query for Ships Database (from Exercise 2.4.3)

**Schema:**
```
Classes(class, type, country, numGuns, bore, displacement)
Ships(name, class, launched)
Battles(name, date)
Outcomes(ship, battle, result)
```

### g) Find the classes that had only one ship as a member of that class.

```sql
SELECT class
FROM Ships
GROUP BY class
HAVING COUNT(*) = 1;
```

Groups ships by class and keeps only those classes where exactly one ship exists; GROUP BY inherently eliminates duplicate class names.

---

## Exercise 6.4.6: Aggregation Queries on Product Database

**Schema:**
```
Product(maker, model, type)
PC(model, speed, ram, hd, price)
Laptop(model, speed, ram, hd, screen, price)
Printer(model, color, type, price)
```

### a) Find the average speed of PC's.

```sql
SELECT AVG(speed)
FROM PC;
```

AVG computes the arithmetic mean of all speed values in the PC table.

### b) Find the average speed of laptops costing over $1000.

```sql
SELECT AVG(speed)
FROM Laptop
WHERE price > 1000;
```

Filters laptops by price before computing the average speed.

### c) Find the average price of PC's made by manufacturer "A."

```sql
SELECT AVG(price)
FROM PC
JOIN Product ON PC.model = Product.model
WHERE maker = 'A';
```

Joins PC with Product to identify manufacturer, then averages the price of A's PCs.

### d) Find the average price of PC's and laptops made by manufacturer "D."

```sql
SELECT AVG(price)
FROM (
    SELECT price FROM PC JOIN Product ON PC.model = Product.model WHERE maker = 'D'
    UNION ALL
    SELECT price FROM Laptop JOIN Product ON Laptop.model = Product.model WHERE maker = 'D'
) AS DProducts;
```

UNION ALL combines prices from both PC and Laptop tables without removing duplicates, ensuring correct average calculation.

### e) Find, for each different speed, the average price of a PC.

```sql
SELECT speed, AVG(price)
FROM PC
GROUP BY speed;
```

GROUP BY partitions PCs by speed, and AVG computes the average price within each group.

### f) Find for each manufacturer, the average screen size of its laptops.

```sql
SELECT maker, AVG(screen)
FROM Product
JOIN Laptop ON Product.model = Laptop.model
GROUP BY maker;
```

Joins Product with Laptop to associate manufacturers with screen sizes, then groups and averages by maker.

### g) Find the manufacturers that make at least three different models of PC.

```sql
SELECT maker
FROM Product
WHERE type = 'pc'
GROUP BY maker
HAVING COUNT(DISTINCT model) >= 3;
```

Groups PC entries by maker and filters those with at least three distinct models using HAVING.

### h) Find for each manufacturer who sells PC's the maximum price of a PC.

```sql
SELECT maker, MAX(price)
FROM Product
JOIN PC ON Product.model = PC.model
GROUP BY maker;
```

Joins to connect makers to PC prices, then finds the maximum price per manufacturer.

### i) Find, for each speed of PC above 2.0, the average price.

```sql
SELECT speed, AVG(price)
FROM PC
WHERE speed > 2.0
GROUP BY speed;
```

Filters PCs with speed above 2.0 before grouping by speed and computing average price.

### j) Find the average hard disk size of a PC for all those manufacturers that make printers.

```sql
SELECT AVG(hd)
FROM PC
JOIN Product ON PC.model = Product.model
WHERE maker IN (
    SELECT maker
    FROM Product
    WHERE type = 'printer'
);
```

The subquery identifies printer manufacturers, and the outer query averages PC hard disk sizes only for those manufacturers.

---

## Exercise 6.4.7: Aggregation Queries on Ships Database

**Schema:**
```
Classes(class, type, country, numGuns, bore, displacement)
Ships(name, class, launched)
Battles(name, date)
Outcomes(ship, battle, result)
```

### a) Find the number of battleship classes.

```sql
SELECT COUNT(*) AS num_battleship_classes
FROM Classes
WHERE type = 'bb';
```

Counts rows in Classes where the type is battleship ('bb').

### b) Find the average number of guns of battleship classes.

```sql
SELECT AVG(numGuns)
FROM Classes
WHERE type = 'bb';
```

Averages numGuns across all battleship class entries, weighting each class equally.

### c) Find the average number of guns of battleships.

```sql
SELECT AVG(numGuns)
FROM Ships
JOIN Classes ON Ships.class = Classes.class
WHERE type = 'bb';
```

Unlike (b), this weights each individual ship equally by joining Ships with Classes, so classes with more ships contribute more to the average.

### d) Find for each class the year in which the first ship of that class was launched.

```sql
SELECT class, MIN(launched) AS first_launched
FROM Ships
GROUP BY class;
```

Groups ships by class and selects the minimum (earliest) launch year per class.

### e) Find for each class the number of ships of that class sunk in battle.

```sql
SELECT class, COUNT(*) AS sunk_count
FROM Ships
JOIN Outcomes ON Ships.name = Outcomes.ship
WHERE result = 'sunk'
GROUP BY class;
```

Joins Ships with Outcomes to find sunk ships, then counts per class.

---

## Exercise 6.5.1: Database Modifications on Product Database

**Schema:**
```
Product(maker, model, type)
PC(model, speed, ram, hd, price)
Laptop(model, speed, ram, hd, screen, price)
Printer(model, color, type, price)
```

### a) Store the fact that PC model 1100 is made by manufacturer C, has speed 3.2, RAM 1024, hard disk 180, and sells for $2499.

```sql
INSERT INTO Product VALUES ('C', 1100, 'pc');
INSERT INTO PC VALUES (1100, 3.2, 1024, 180, 2499);
```

Two INSERT statements are needed because the data spans two tables: Product (for manufacturer info) and PC (for specifications).

### b) Insert laptops for every PC with the same manufacturer, speed, RAM, and hard disk, a 17-inch screen, model number 1100 greater, and price $500 more.

```sql
INSERT INTO Product
    SELECT maker, PC.model + 1100, 'laptop'
    FROM Product JOIN PC ON Product.model = PC.model;

INSERT INTO Laptop
    SELECT model + 1100, speed, ram, hd, 17, price + 500
    FROM PC;
```

The first INSERT creates Product entries for the new laptops; the second populates the Laptop table with computed attributes from existing PCs.

### c) Delete all PC's with less than 100 gigabytes of hard disk.

```sql
DELETE FROM PC
WHERE hd < 100;
```

Removes rows from PC where the hard disk size is below 100.

### d) Delete all laptops made by a manufacturer that doesn't make printers.

```sql
DELETE FROM Laptop
WHERE model IN (
    SELECT model
    FROM Product
    WHERE maker NOT IN (
        SELECT maker
        FROM Product
        WHERE type = 'printer'
    )
    AND type = 'laptop'
);
```

The inner subquery finds manufacturers without printers, and the outer query deletes laptops from those manufacturers.

### e) Manufacturer A buys manufacturer B. Change all products made by B so they are now made by A.

```sql
UPDATE Product
SET maker = 'A'
WHERE maker = 'B';
```

A simple UPDATE changes the maker attribute for all of B's products to A.

### f) For each PC, double the amount of RAM and add 60 gigabytes to the hard disk.

```sql
UPDATE PC
SET ram = ram * 2, hd = hd + 60;
```

Multiple attributes can be updated in a single SET clause, separated by commas.

### g) For each laptop made by manufacturer B, add one inch to the screen size and subtract $100 from the price.

```sql
UPDATE Laptop
SET screen = screen + 1, price = price - 100
WHERE model IN (
    SELECT model
    FROM Product
    WHERE maker = 'B'
);
```

The subquery identifies B's laptop models, and the UPDATE modifies screen and price for those models.
