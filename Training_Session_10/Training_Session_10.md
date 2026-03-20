## Exercise 7.5.1

The given example trigger fires **after an UPDATE** and rolls back the change if the average `netWorth` would drop below 500 000. Analogous triggers for **INSERT** and **DELETE** follow the same pattern: reference the rows that were added or removed, check the same condition, and undo the modification.

---

### UPDATE trigger

Since PostgreSQL does not support logic in triggers, we need to write a function for the trigger to run, after "catching" the update. Thus here the trigger from the exercise rewritten to run in postgresql. 

```sql
CREATE FUNCTION CheckNetworthAverage()
	RETURNS TRIGGER AS $$
	BEGIN
		RAISE NOTICE 'Update triggered! Actual average: %', (SELECT AVG(netWorth) FROM MovieExec);
		IF 500000 > (SELECT AVG(netWorth) FROM MovieExec) THEN
			RAISE NOTICE 'Average below 500000. Reverting change...';
			DELETE FROM MovieExec
			WHERE (cert) IN 
				(SELECT cert FROM NewStuff);
			INSERT INTO MovieExec
				(SELECT * FROM OldStuff);	
		END IF;
		RETURN NULL;
	END
	$$ LANGUAGE plpgsql;


CREATE TRIGGER AvgNetWorthTriggerUpdatePSQL
	AFTER UPDATE ON MovieExec
	REFERENCING
		OLD TABLE AS OldStuff
		NEW TABLE AS NewStuff
	FOR EACH STATEMENT
	EXECUTE PROCEDURE
		CheckNetworthAverage();
```

---

### INSERT trigger

After inserting new rows into `MovieExec`, if the new average `netWorth` is too low, delete the just-inserted rows (identified via `NEW TABLE`).

```sql
CREATE OR REPLACE FUNCTION CheckNetworthAfterInsert()
RETURNS trigger AS $$
BEGIN
   IF 500000 > (SELECT AVG(netWorth) FROM MovieExec) THEN
      DELETE FROM MovieExec
      WHERE (name, address, cert, netWorth) IN
         (SELECT name, address, cert, netWorth FROM NewStuff);
   END IF;
   RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER AvgNetWorthInsertTrigger
AFTER INSERT ON MovieExec
REFERENCING NEW TABLE AS NewStuff
FOR EACH STATEMENT
EXECUTE FUNCTION CheckNetworthAfterInsert();
```

---

### DELETE trigger

After deleting rows from `MovieExec`, if the new average `netWorth` is too low, re-insert the just-deleted rows (identified via `OLD TABLE`).

```sql
CREATE OR REPLACE FUNCTION CheckNetworthAfterDelete()
RETURNS trigger AS $$
BEGIN
   IF 500000 > (SELECT AVG(netWorth) FROM MovieExec) THEN
      INSERT INTO MovieExec
         SELECT * FROM OldStuff;
   END IF;
   RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER AvgNetWorthDeleteTrigger
AFTER DELETE ON MovieExec
REFERENCING OLD TABLE AS OldStuff
FOR EACH STATEMENT
EXECUTE FUNCTION CheckNetworthAfterDelete();
```

---

## Exercise 7.4.1

Assertions are global constraints declared with `CREATE ASSERTION ... CHECK (...)`. The condition must hold after every modification; if it is violated the modification is rejected.

---

### a) A manufacturer who makes PCs must not also make laptops.

Find all makers who appear as both a PC maker and a laptop maker and assert that set is empty.

```sql
CREATE ASSERTION NoPCAndLaptopMaker
CHECK (NOT EXISTS (
   SELECT maker FROM Product WHERE type = 'pc'
   INTERSECT
   SELECT maker FROM Product WHERE type = 'laptop'
));
```

---

### b) A manufacturer of a PC must also make a laptop with at least the same processor speed.

For every (maker, PC-speed) pair in the database there must exist at least one laptop from the same maker whose speed is >= that PC's speed.

```sql
CREATE ASSERTION PCMakerMakesEquivalentLaptop
CHECK (NOT EXISTS (
   SELECT p.maker, pc.speed
   FROM Product p JOIN PC pc ON p.model = pc.model
   WHERE NOT EXISTS (
      SELECT 1
      FROM Product p2 JOIN Laptop l ON p2.model = l.model
      WHERE p2.maker = p.maker
        AND l.speed >= pc.speed
   )
));
```

---

### c) If a laptop has more RAM than a PC, then the laptop must also have a higher price.

There must be no (laptop, PC) pair where the laptop has more RAM but not a higher price.

```sql
CREATE ASSERTION LaptopRAMImpliesHigherPrice
CHECK (NOT EXISTS (
   SELECT 1
   FROM Laptop l, PC p
   WHERE l.ram > p.ram
     AND l.price <= p.price
));
```

---

### d) If a model appears in Product, it must appear in the corresponding table for its type.

Every model whose type is `'pc'` must be in `PC`, every `'laptop'` model must be in `Laptop`, and every `'printer'` model must be in `Printer`.

```sql
CREATE ASSERTION ModelAppearsInCorrectTable
CHECK (
   NOT EXISTS (
      SELECT model FROM Product
      WHERE type = 'pc' AND model NOT IN (SELECT model FROM PC)
   )
   AND NOT EXISTS (
      SELECT model FROM Product
      WHERE type = 'laptop' AND model NOT IN (SELECT model FROM Laptop)
   )
   AND NOT EXISTS (
      SELECT model FROM Product
      WHERE type = 'printer' AND model NOT IN (SELECT model FROM Printer)
   )
);
```

---

## Exercise 7.5.2

Triggers can enforce constraints by undoing a modification when it violates the constraint. The pattern is: fire `AFTER` the event, reference the affected rows, check the condition in a `WHEN` clause, and roll back by reversing the change.

---

### a) When updating the price of a PC, check that there is no lower-priced PC with the same speed.

After the update, look for any newly-updated row whose new price is higher than another PC with the same speed. If such a row exists, restore the old prices by deleting the updated rows and re-inserting the originals.

```sql
CREATE OR REPLACE FUNCTION CheckLowerPricedPCWithSameSpeed()
RETURNS trigger AS $$
BEGIN
   IF EXISTS (
      SELECT 1
      FROM NewStuff n, PC p
      WHERE n.speed = p.speed
        AND p.price < n.price
        AND p.model NOT IN (SELECT model FROM NewStuff)
   ) THEN
      DELETE FROM PC
      WHERE (model, speed, ram, hd, price) IN
         (SELECT model, speed, ram, hd, price FROM NewStuff);
      INSERT INTO PC
         SELECT * FROM OldStuff;
   END IF;
   RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER PCPriceUpdateCheck
AFTER UPDATE OF price ON PC
REFERENCING OLD TABLE AS OldStuff NEW TABLE AS NewStuff
FOR EACH STATEMENT
EXECUTE FUNCTION CheckLowerPricedPCWithSameSpeed();
```

---

### b) When inserting a new printer, check that the model number exists in Product.

After the insert, if any newly-inserted printer's model does not exist in `Product`, delete those rows to undo the insert.

```sql
CREATE OR REPLACE FUNCTION CheckPrinterInProductInsert()
RETURNS trigger AS $$
BEGIN
   IF EXISTS (
      SELECT 1 FROM NewStuff
      WHERE model NOT IN (SELECT model FROM Product)
   ) THEN
      DELETE FROM Printer
      WHERE (model, color, type, price) IN
         (SELECT model, color, type, price FROM NewStuff);
   END IF;
   RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER PrinterInsertCheck
AFTER INSERT ON Printer
REFERENCING NEW TABLE AS NewStuff
FOR EACH STATEMENT
EXECUTE FUNCTION CheckPrinterInProductInsert();
```
