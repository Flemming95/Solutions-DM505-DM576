# Training Session 1 - Database Systems Week 6

## Course Information

**Course book:** Garcia-Molina, Ullman, Widom; *Database Systems: The Complete Book*, 2nd Ed. 2014

### Lecture 1: February 3
- **Reading:** Chapters 1, 2.1-2.3
- **Topics:** Course Organization, Databases, Relational Data Model

### Lecture 2: February 4
- **Reading:** Chapters 2.4, 5.1
- **Topics:** Relational Algebra including its basic operations
- **Exercises:** 2.2.1, 2.2.2, 2.2.3, 2.3.1, 2.3.2

---

## Exercise Solutions

### Exercise 2.1: Banking Database Analysis

Based on Figure 6 showing instances of two relations (Accounts and Customers) from a banking database:

#### a) The attributes of each relation

| Relation   | Attributes                          |
|------------|-------------------------------------|
| Accounts   | acctNo, type, balance               |
| Customers  | firstName, lastName, idNo, account  |

#### b) The tuples of each relation

**Accounts relation tuples:**
- (12345, savings, 12000)
- (23456, checking, 1000)
- (34567, savings, 25)

**Customers relation tuples:**
- (Robbie, Banks, 901-222, 12345)
- (Lena, Hand, 805-333, 12345)
- (Lena, Hand, 805-333, 23456)

#### c) The components of one tuple from each relation

**From Accounts:** For tuple (12345, savings, 12000):
- acctNo component: 12345
- type component: savings
- balance component: 12000

**From Customers:** For tuple (Robbie, Banks, 901-222, 12345):
- firstName component: Robbie
- lastName component: Banks
- idNo component: 901-222
- account component: 12345

#### d) The relation schema for each relation

```
Accounts(acctNo, type, balance)
Customers(firstName, lastName, idNo, account)
```

#### e) The database schema

```
{
    Accounts(acctNo, type, balance),
    Customers(firstName, lastName, idNo, account)
}
```

#### f) A suitable domain for each attribute

| Attribute  | Domain                                          |
|------------|-------------------------------------------------|
| acctNo     | INT                                |
| type       | {'savings', 'checking'}                         |
| balance    | Non-negative real numbers (monetary values)     |
| firstName  | VARCHAR(50)                                     |
| lastName   | VARCHAR(50)                                     |
| idNo       | CHAR(7) (because we have a defined pattern)     |
| account    | 5-digit integers (references acctNo in Accounts)|

#### g) Another equivalent way to present each relation

**Accounts (reordered attributes and tuples):**

| balance | acctNo | type     |
|---------|--------|----------|
| 1000    | 23456  | checking |
| 25      | 34567  | savings  |
| 12000   | 12345  | savings  |

**Customers (reordered attributes and tuples):**

| lastName | firstName | account | idNo    |
|----------|-----------|---------|---------|
| Hand     | Lena      | 23456   | 805-333 |
| Banks    | Robbie    | 12345   | 901-222 |
| Hand     | Lena      | 12345   | 805-333 |

---

**Alternatively, these could also be represented as an ER-Diagram.**

### Exercise 2.2: Examples of Key Attributes

Attributes that are created specifically to serve as keys of relations:

1. **Social Security Number (SSN)** - Created to uniquely identify individuals in the United States
2. **ISBN (International Standard Book Number)** - Unique identifier for books
3. **VIN (Vehicle Identification Number)** - Unique 17-character code for motor vehicles
4. **Employee ID** - Company-assigned unique identifier for employees
5. **Student ID** - University-assigned unique identifier for students
6. **Order Number** - E-commerce systems generate unique order identifiers
7. **Transaction ID** - Banking systems assign unique identifiers to each transaction
8. **SKU (Stock Keeping Unit)** - Retail inventory tracking codes
9. **UUID (Universally Unique Identifier)** - 128-bit identifiers used in distributed systems
10. **Passport Number** - Government-issued unique identifier for travelers
11. **License Plate Number** - Unique identifier for registered vehicles
12. **IBAN (International Bank Account Number)** - Unique identifier for bank accounts internationally

---

### Exercise 2.3: Counting Ways to Represent Relations

#### a) Three attributes and three tuples

**Number of attribute orderings:** 3! = 6

**Number of tuple orderings:** 3! = 6

**Total different representations:** 3! × 3! = 6 × 6 = **36 ways**

#### b) Four attributes and five tuples

**Number of attribute orderings:** 4! = 24

**Number of tuple orderings:** 5! = 120

**Total different representations:** 4! × 5! = 24 × 120 = **2,880 ways**

#### c) n attributes and m tuples

**Number of attribute orderings:** n!

**Number of tuple orderings:** m!

**Total different representations:** **n! × m! ways**

---

### Exercise 3.1: Product Database Schema Declarations

#### a) Schema for relation Product

```sql
CREATE TABLE Product (
    maker     CHAR(20),
    model     CHAR(10) PRIMARY KEY,
    type      CHAR(10)
);
```

#### b) Schema for relation PC

```sql
CREATE TABLE PC (
    model   CHAR(10) PRIMARY KEY,
    speed   DECIMAL(4,2),
    ram     INT,
    hd      INT,
    price   DECIMAL(10,2),
);
```

#### c) Schema for relation Laptop

```sql
CREATE TABLE Laptop (
    model   CHAR(10) PRIMARY KEY,
    speed   DECIMAL(4,2),
    ram     INT,
    hd      INT,
    screen  DECIMAL(3,1),
    price   DECIMAL(10,2),
);
```

#### d) Schema for relation Printer

```sql
CREATE TABLE Printer (
    model   CHAR(10) PRIMARY KEY,
    color   BOOLEAN,
    type    CHAR(10),
    price   DECIMAL(10,2),
);
```

#### e) Alteration to delete attribute color from Printer

```sql
ALTER TABLE Printer DROP COLUMN color;
```

#### f) Alteration to add attribute od to Laptop with default value

```sql
ALTER TABLE Laptop ADD COLUMN od CHAR(10) DEFAULT 'none';
```

---

### Exercise 3.2: World War II Capital Ships Schema Declarations

#### a) Schema for relation Classes

```sql
CREATE TABLE Classes (
    class         CHAR(20) PRIMARY KEY,
    type          CHAR(2),
    country       CHAR(20),
    numGuns       INT,
    bore          DECIMAL(3,1),
    displacement  INT
);
```

#### b) Schema for relation Ships

```sql
CREATE TABLE Ships (
    name      CHAR(30) PRIMARY KEY,
    class     CHAR(20),
    launched  INT,
);
```

#### c) Schema for relation Battles

```sql
CREATE TABLE Battles (
    name  CHAR(30) PRIMARY KEY,
    date  DATE
);
```

#### d) Schema for relation Outcomes
**I used a primary key consisting of ship+battle**

```sql
CREATE TABLE Outcomes (
    ship    CHAR(30),
    battle  CHAR(30),
    result  CHAR(10),
    PRIMARY KEY (ship, battle),
);
```

#### e) Alteration to delete attribute bore from Classes

```sql
ALTER TABLE Classes DROP COLUMN bore;
```

#### f) Alteration to add attribute yard to Ships

```sql
ALTER TABLE Ships ADD COLUMN yard CHAR(30);
```

---

## Summary

This training session covers fundamental concepts in database systems:

| Topic | Key Concepts |
|-------|--------------|
| Relational Data Model | Relations, attributes, tuples, domains, schemas |
| Keys | Unique identifiers, primary keys, foreign keys |
| Relation Representation | Multiple equivalent representations (n! × m! ways) |
| SQL DDL | CREATE TABLE, ALTER TABLE, data types, constraints |
