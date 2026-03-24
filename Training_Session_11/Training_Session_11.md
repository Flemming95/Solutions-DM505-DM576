# Training Session 11

## Exercise 6.6.1

Transactions group SQL statements so they either all succeed or all fail. `BEGIN TRANSACTION` starts the unit of work, `COMMIT` makes changes permanent, and `ROLLBACK` undoes everything if something goes wrong. A read-only transaction cannot change the database, so we add `SET TRANSACTION READ ONLY` to let the DBMS optimise concurrency.

---

### a) Look up PCs by speed and RAM, print model and price

This is a **read-only** transaction — it only reads data, so no changes need to be rolled back.

#### Pseudo-code

```
FUNCTION lookupPC(p_speed IN, p_ram IN):

    SET TRANSACTION READ ONLY
    BEGIN TRANSACTION

    SELECT model, price
    FROM   PC
    WHERE  speed = p_speed
      AND  ram   = p_ram

    FOR EACH row IN result:
        PRINT row.model, row.price

    COMMIT
```

#### Real PostgreSQL

```sql
CREATE OR REPLACE FUNCTION lookupPC(p_speed numeric, p_ram INT)
RETURNS VOID AS
$$
DECLARE
    rec RECORD;
BEGIN


    FOR rec IN
        SELECT model, price
        FROM PC
        WHERE speed = p_speed
          AND ram   = p_ram
    LOOP
        RAISE NOTICE 'Model: %, Price: %', rec.model, rec.price;
    END LOOP;

END;
$$ LANGUAGE plpgsql;

SELECT lookupPC(2.2, 1024)
```

---

### b) Delete a model from PC and Product

Both deletes must succeed together. If the second delete fails, `ROLLBACK` undoes the first, so we never end up with a dangling `Product` row or an orphan `PC` row.

#### Pseudo-code

```
FUNCTION deleteModel(model_in IN):

    BEGIN TRANSACTION

    DELETE FROM PC
    WHERE  model = model_in

    DELETE FROM Product
    WHERE  model = model_in

    IF error:
        ROLLBACK
    ELSE:
        COMMIT
```

#### Real PostgreSQL

```sql
CREATE OR REPLACE FUNCTION delete_model(model_in INT)
RETURNS VOID AS $$
BEGIN
    DELETE FROM PC
    WHERE  model = model_in;

    DELETE FROM Product
    WHERE  model = model_in;

EXCEPTION WHEN OTHERS THEN
    RAISE; -- raise will automatically rollback all changses.
END;
$$ LANGUAGE plpgsql;

SELECT delete_model('1002');
```

---

### c) Decrease the price of a PC by $100

A single `UPDATE`. The transaction ensures the change is either fully applied or fully undone — no half-updated price is ever visible.

#### Pseudo-code

```
FUNCTION decreasePrice(model_in IN):

    BEGIN TRANSACTION

    UPDATE PC
    SET    price = price - 100
    WHERE  model = model_in

    IF error:
        ROLLBACK
    ELSE:
        COMMIT
```

#### Real PostgreSQL

```sql
-- Create a function that decreases the price of the given model by $100
CREATE OR REPLACE FUNCTION decrease_price(model_in INT)
RETURNS VOID AS $$
BEGIN
    UPDATE PC
    SET    price = price - 100
    WHERE  model = model_in;

EXCEPTION WHEN OTHERS THEN
    RAISE;
END;
$$ LANGUAGE plpgsql;

SELECT decrease_price('1003');
```

---

### d) Insert a new PC model into PC and Product (if it does not exist)

First we check for a duplicate, then insert into both tables. Both inserts must be atomic — a crash between them would leave the database inconsistent.

#### Pseudo-code

```
FUNCTION addPC(maker_in, model_in, speed_in, ram_in, hd_in, price_in):

    BEGIN TRANSACTION

    -- Check for existing model
    SELECT COUNT(*) INTO cnt
    FROM   Product
    WHERE  model = model_in

    IF cnt > 0:
        PRINT "Error: model already exists."
        ROLLBACK
        RETURN

    -- Insert into Product, then into PC
    INSERT INTO Product(maker, model, type)
    VALUES (maker_in, model_in, 'PC')

    INSERT INTO PC(model, speed, ram, hd, price)
    VALUES (model_in, speed_in, ram_in, hd_in, price_in)

    IF error:
        ROLLBACK
    ELSE:
        COMMIT
```

#### Real PostgreSQL

```sql
CREATE OR REPLACE FUNCTION add_pc(
    maker_in  CHAR(10),
    model_in  INT,
    speed_in  NUMERIC,
    ram_in    INT,
    hd_in     INT,
    price_in  INT
)
RETURNS VOID AS $$
DECLARE
    cnt INTEGER;
BEGIN
    SELECT COUNT(*) INTO cnt
    FROM   Product
    WHERE  model = model_in;

    IF cnt > 0 THEN
        RAISE EXCEPTION 'Error: model % already exists.', model_in;
    END IF;

    INSERT INTO Product(maker, model, type)
    VALUES (maker_in, model_in, 'PC');

    INSERT INTO PC(model, speed, ram, hd, price)
    VALUES (model_in, speed_in, ram_in, hd_in, price_in);

EXCEPTION WHEN OTHERS THEN
    RAISE;
END;
$$ LANGUAGE plpgsql;

SELECT add_pc('A', 1030, 3.0, 1024, 500, 800);
```

---

## Exercise 6.6.2

If the system crashes in the middle of a transaction, the database may be left in an inconsistent state — unless the transaction mechanism rolls it back on restart.

---

### a) Lookup PCs (read-only)

No atomicity problem. The transaction only reads data; a crash leaves nothing written to the database.

---

### b) Delete from PC and Product

If the system crashes after deleting from `PC` but before deleting from `Product`, the `Product` table still contains a model for which no `PC` row exists (a "ghost" product). Conversely, if it crashes after deleting from `Product` first, we have a `PC` row with no matching `Product` entry. The transaction and rollback-on-restart prevents both scenarios, because neither delete is committed until both succeed.

---

### c) Decrease price by $100

If the system crashes mid-`UPDATE` (extremely unlikely at the SQL level, but possible at the page-write level), the price could be partially written. The transaction guarantees the full update is applied or not at all — on restart the DBMS rolls back the incomplete change and the original price is restored.

---

### d) Insert into Product and PC

If the system crashes after inserting into `Product` but before inserting into `PC`, there is a product with no PC details — a referential consistency violation (assuming a foreign key from `PC.model` to `Product.model`). The transaction and rollback ensures either both inserts are committed or neither is.

---

## Exercise 6.6.3

`READ UNCOMMITTED` lets a transaction read data written by other transactions that have **not yet committed** (a *dirty read*). `SERIALIZABLE` guarantees that the outcome is equivalent to running the transactions one at a time in some order, so dirty reads, non-repeatable reads, and phantom reads are all prevented.

---

### a) Lookup PCs (read-only)

Under `READ UNCOMMITTED`, T might read a price or RAM value that another concurrent transaction has changed but not yet committed. If that other transaction is later rolled back, T has seen data that never "officially" existed. Under `SERIALIZABLE`, T is guaranteed to see only committed values, so this cannot happen.

---

### b) Delete from PC and Product

Under `READ UNCOMMITTED`, another transaction T' executing the same delete might have already deleted the `PC` row (but not yet committed). T would find no row to delete from `PC` and then also try to delete from `Product`, potentially resulting in an inconsistent state if T' later rolls back. Under `SERIALIZABLE`, only one of the transactions runs the delete sequence at a time, so this conflict cannot arise.

---

### c) Decrease price by $100

Under `READ UNCOMMITTED`, if two transactions both execute the price decrease on the same model concurrently, they may both read the same original price and both write `original_price − 100`, effectively applying the decrease only once instead of twice (the **lost update** problem). Under `SERIALIZABLE`, the two updates are ordered, so the price is correctly reduced by $200 in total.

---

### d) Insert a new PC model

Under `READ UNCOMMITTED`, two concurrent transactions running program (d) with the same model number could both read `COUNT(*) = 0` (before either has committed its insert) and both proceed to insert — resulting in a duplicate-model violation or two conflicting rows. Under `SERIALIZABLE`, the check and the insert are treated as an atomic sequence, so only one transaction can succeed with a given new model number.
