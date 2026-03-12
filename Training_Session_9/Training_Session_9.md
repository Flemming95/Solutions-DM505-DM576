# Training Session 9

## Exercise 7.1.1

**Referential integrity** means that a value in a foreign key column must either be NULL or match a primary key value in the referenced table. We declare this in SQL using `FOREIGN KEY ... REFERENCES`, and we control what happens on a violation using `ON DELETE` and `ON UPDATE` actions:

- `NO ACTION` (default) — reject the modification that would create a dangling reference.
- `SET NULL` — set the foreign key column to NULL when the referenced row is deleted or updated.
- `CASCADE` — propagate the deletion or update to all referencing rows.

---

### a) The producer of a movie must be someone mentioned in MovieExec. Violations are rejected.

`Movies.producerC` references `MovieExec.cert`. The default action `NO ACTION` rejects any deletion or update of a `MovieExec` row that would leave a movie without a valid producer.

```sql
-- Drop existing tables to avoid conflicts
DROP TABLE Movies;
DROP TABLE movieexec;

-- Create tables according to task
CREATE TABLE movieexec (
    name char(30),
    address varchar(255),
    cert INT,
    networth INT,
    PRIMARY KEY (cert)
);

CREATE TABLE Movies (
    title       VARCHAR(255),
    year        INT,
    length      INT,
    genre       VARCHAR(30),
    studioName  VARCHAR(55),
    producerC  INT,
    PRIMARY KEY (title, year),
    FOREIGN KEY (producerC) REFERENCES MovieExec(cert)
        ON DELETE NO ACTION
        ON UPDATE NO ACTION
);
```

---

### b) Repeat (a), but violations set producerC# in Movies to NULL.

When the referenced `MovieExec` row is deleted or its `cert` is updated, the `producerC` in every affected `Movies` row is set to `NULL` instead of causing an error.

```sql
-- Drop existing tables to avoid conflicts
DROP TABLE Movies;

CREATE TABLE Movies (
    title       VARCHAR(255),
    year        INT,
    length      INT,
    genre       VARCHAR(50),
    studioName  VARCHAR(100),
    producerC  INT,
    PRIMARY KEY (title, year),
    FOREIGN KEY (producerC) REFERENCES MovieExec(cert)
        ON DELETE SET NULL
        ON UPDATE SET NULL
);
```

---

### c) Repeat (a), but violations delete or update the offending Movies tuple.

When the referenced `MovieExec` row is deleted or updated, every `Movies` row that references it is also deleted or updated to match.

```sql
-- Drop existing tables to avoid conflicts
DROP TABLE Movies;

CREATE TABLE Movies (
    title       VARCHAR(255),
    year        INT,
    length      INT,
    genre       VARCHAR(50),
    studioName  VARCHAR(100),
    producerC#  INT,
    PRIMARY KEY (title, year),
    FOREIGN KEY (producerC#) REFERENCES MovieExec(cert#)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

---

### d) A movie in StarsIn must also appear in Movies. Violations are rejected.

`StarsIn.(movieTitle, movieYear)` is a composite foreign key referencing `Movies.(title, year)`. Rejecting the modification means using `NO ACTION`.

```sql
DROP TABLE StarsIn;

CREATE TABLE StarsIn (
    movieTitle  VARCHAR(255),
    movieYear   INT,
    starName    VARCHAR(255),
    PRIMARY KEY (movieTitle, movieYear, starName),
    FOREIGN KEY (movieTitle, movieYear) REFERENCES Movies(title, year)
        ON DELETE NO ACTION
        ON UPDATE NO ACTION
);
```

---

### e) A star in StarsIn must also appear in MovieStar. Violations delete violating tuples.

`StarsIn.starName` references `MovieStar.name`. If a `MovieStar` row is deleted, the corresponding `StarsIn` rows are automatically deleted as well (`CASCADE`).

```sql
DROP TABLE StarsIn;
DROP TABLE MovieStar;

CREATE TABLE MovieStar (
	name VARCHAR(255),
	address VARCHAR(255),
	gender VARCHAR(50),
	netWorth INT,
	PRIMARY KEY (name)
);

CREATE TABLE StarsIn (
    movieTitle  VARCHAR(255),
    movieYear   INT,
    starName    VARCHAR(255),
    PRIMARY KEY (movieTitle, movieYear, starName),
    FOREIGN KEY (starName) REFERENCES MovieStar(name)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

---

## Exercise 7.1.3

**Key** — a minimal set of attributes that uniquely identifies every row.  
**Foreign key** — one or more attributes whose values must match a primary key in another (or the same) table.

For the PC database:

- `Product.model` is the natural primary key — every product has a unique model number.
- `PC`, `Laptop`, and `Printer` each describe a specific model, so `model` is also their primary key and simultaneously a foreign key back to `Product`.

```sql
ALTER TABLE Product (
    ADD PRIMARY KEY (model)
);

ALTER TABLE PC (
    ADD PRIMARY KEY (model),
    ADD CONSTRAINT fk_PC_product --fk: foreign key
        FOREIGN KEY (model)
        REFERENCES Product(model)
);

ALTER TABLE Laptop (
    ADD PRIMARY KEY (model),
    ADD CONSTRAINT fk_Laptop_Product
        FOREIGN KEY (model)
        REFERENCES Product(model)
);

CREATE TABLE Printer (
    ADD PRIMARY KEY (model),
    ADD CONSTRAINT fk_PC_Product
        FOREIGN KEY (model)
        REFERENCES Product(model)
);
```

---

## Exercise 7.2.1

**Attribute constraints** are declared with the `CHECK` keyword directly on a column. The constraint is evaluated whenever a row is inserted or updated; if the condition is false the modification is rejected.

---

### a) The year cannot be before 1915.

```sql
year INT CHECK (year >= 1915)
```

---

### b) The length of a movie must be neither less than 60 nor more than 250.

```sql
length INT CHECK (length >= 60 AND length <= 250)
```

---

### c) The studio name can only be Disney, Fox, MGM, or Paramount.

```sql
studioName VARCHAR(100) CHECK (studioName IN ('Disney', 'Fox', 'MGM', 'Paramount'))
-- OR
studioName VARCHAR(100) CHECK(
		studioName = 'Disney' OR
		studioName = 'Fox' OR
		studioName = 'MGM' OR
		studioName = 'Paramount'
	);
```

Full table declaration combining all three constraints:

```sql
CREATE TABLE Movies (
    title       VARCHAR(255),
    year        INT          CHECK (year >= 1915),
    length      INT          CHECK (length >= 60 AND length <= 250),
    genre       VARCHAR(50),
    studioName  VARCHAR(100) CHECK (studioName IN ('Disney', 'Fox', 'MGM', 'Paramount')),
    producerC#  INT,
    PRIMARY KEY (title, year)
);
```
