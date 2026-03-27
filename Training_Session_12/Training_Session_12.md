# Training Session 12

## Exercise 9.4.1

**PSM** (Persistent Stored Modules) lets you write stored procedures and functions that run inside the DBMS. Use `CREATE PROCEDURE` for routines with `OUT` parameters or side effects, and `CREATE FUNCTION` for routines that return a single value via `RETURN`. Local variables are declared with `DECLARE` and assigned with `SET` or `SELECT ... INTO`.

---

### a) Net worth of studio president given studio name

Join `Studio` (which has `presC`) with `MovieExec` (which has `cert` and `netWorth`) and select the result into a local variable.

```sql
CREATE OR REPLACE FUNCTION studioPresidentNetWorth(studioName VARCHAR)
RETURNS INT AS $$
DECLARE
    worth INT;
BEGIN
    SELECT me.networth
    INTO worth
    FROM Studio s
    JOIN MovieExec me ON s.presC = me.cert
    WHERE s.name = studioName;

    RETURN worth;
END;
$$ LANGUAGE plpgsql;
```
ALTERNATIVELY you can also choose to return the value directly
```sql
CREATE OR REPLACE FUNCTION studioPresidentNetWorth(studioName VARCHAR)
RETURNS INT AS $$
BEGIN
    RETURN (
        SELECT me.networth
        FROM Studio s
        JOIN MovieExec me ON s.presC = me.cert
        WHERE s.name = studioName
        LIMIT 1
    );
END;
$$ LANGUAGE plpgsql;
```
---

### b) Return 1 (star only), 2 (exec only), 3 (both), 4 (neither)

Use `COUNT(*)` into two flag variables, then branch with `IF/ELSEIF` on the combination.

```sql
CREATE OR REPLACE FUNCTION personCategory(pName VARCHAR, pAddress VARCHAR)
RETURNS INT AS $$
DECLARE
    isStar INT := 0;
    isExec INT := 0;
BEGIN
    SELECT COUNT(*) INTO isStar
    FROM MovieStar
    WHERE name = pName AND address = pAddress;

    SELECT COUNT(*) INTO isExec
    FROM MovieExec
    WHERE name = pName AND address = pAddress;

    IF isStar > 0 AND isExec = 0 THEN
        RETURN 1;
    ELSIF isStar = 0 AND isExec > 0 THEN
        RETURN 2;
    ELSIF isStar > 0 AND isExec > 0 THEN
        RETURN 3;
    ELSE
        RETURN 4;
    END IF;
END;
$$ LANGUAGE plpgsql;
```

---

### c) Titles of the two longest movies for a studio (OUT parameters)

Initialize both `OUT` parameters to `NULL`. Use `ORDER BY length DESC LIMIT 1` for the longest and `LIMIT 1 OFFSET 1` for the second. If there are fewer than 2 movies the second (or both) parameters stay `NULL`.

```sql
CREATE OR REPLACE PROCEDURE twoLongestMovies(
    IN studio VARCHAR,
    OUT title1 VARCHAR,
    OUT title2 VARCHAR
)
LANGUAGE plpgsql
AS $$
DECLARE
    movieCount INT;
BEGIN
    title1 := NULL;
    title2 := NULL;

    SELECT COUNT(*) INTO movieCount
    FROM Movies
    WHERE studioName = studio;

    IF movieCount >= 1 THEN
        SELECT title INTO title1
        FROM Movies
        WHERE studioName = studio
        ORDER BY length DESC
        LIMIT 1;
    END IF;

    IF movieCount >= 2 THEN
        SELECT title INTO title2
        FROM Movies
        WHERE studioName = studio
        ORDER BY length DESC
        LIMIT 1 OFFSET 1;
    END IF;
END;
$$;
```

---

### d) Earliest movie > 120 minutes for a star; return 0 if none

Join `StarsIn` with `Movies` on both title and year, filter by the star name and `length > 120`, then take `MIN(year)`. `MIN` returns `NULL` when there are no matching rows, so check for that and return `0` instead.

```sql
CREATE OR REPLACE FUNCTION earliestLongMovie(sName VARCHAR)
RETURNS INT AS $$
DECLARE
    minYear INT;
BEGIN
    SELECT MIN(m.year)
    INTO minYear
    FROM Movies m
    JOIN StarsIn si
      ON m.title = si.movieTitle
     AND m.year  = si.movieYear
    WHERE si.starName = sName
      AND m.length > 120;

    IF minYear IS NULL THEN
        RETURN 0;
    END IF;

    RETURN minYear;
END;
$$ LANGUAGE plpgsql;
```

---

### e) Return the unique star at an address, or NULL if none or more than one

Count how many stars share the address. If exactly one, fetch the name; otherwise return `NULL`.

```sql
CREATE OR REPLACE FUNCTION uniqueStarAtAddress(addr VARCHAR)
RETURNS VARCHAR AS $$
DECLARE
    starName VARCHAR;
    cnt INT;
BEGIN
    SELECT COUNT(*) INTO cnt
    FROM MovieStar
    WHERE address = addr;

    IF cnt = 1 THEN
        SELECT name INTO starName
        FROM MovieStar
        WHERE address = addr;

        RETURN starName;
    ELSE
        RETURN NULL;
    END IF;
END;
$$ LANGUAGE plpgsql;
```

---

### f) Delete a star, all their StarsIn rows, and all movies they appeared in

Save the star's movies to a temporary table first, so the list is preserved after deleting from `StarsIn`. Delete from `StarsIn` before `Movies` to satisfy foreign-key constraints, then remove the star from `MovieStar`.

```sql
CREATE OR REPLACE PROCEDURE deleteStar(sName VARCHAR)
LANGUAGE plpgsql
AS $$
BEGIN
    -- Create temporary table
    CREATE TEMP TABLE starMovies AS
        SELECT movieTitle, movieYear
        FROM StarsIn
        WHERE starName = sName;

    -- Delete dependent rows first (FK safety)
    DELETE FROM StarsIn
    WHERE (movieTitle, movieYear) IN (
        SELECT movieTitle, movieYear FROM starMovies
    );

    -- Delete movies
    DELETE FROM Movies
    WHERE (title, year) IN (
        SELECT movieTitle, movieYear FROM starMovies
    );

    -- Drop temp table
    DROP TABLE starMovies;

    -- Delete the star
    DELETE FROM MovieStar
    WHERE name = sName;
END;
$$;
```

---

## Exercise 9.3.1

**Embedded SQL** mixes SQL statements directly into a host-language program. Host variables are declared inside a `DECLARE SECTION` and referenced with a `:` prefix inside SQL. Single-row results use `SELECT ... INTO :var`; multi-row results require a **cursor** (declare → open → fetch loop → close). The examples below use C-style syntax; `SQLSTATE == "02000"` signals "no more rows".

---

### a) Find the PC whose price is closest to the user's desired price

The subquery computes the minimum absolute price difference; the outer query selects the PC that matches it.

```c
EXEC SQL BEGIN DECLARE SECTION;
    double desiredPrice;
    char   maker[30], model[10];
    double speed;
EXEC SQL END DECLARE SECTION;

printf("Enter desired price: ");
scanf("%lf", &desiredPrice);

EXEC SQL
    SELECT p.maker, pc.model, pc.speed
    INTO   :maker, :model, :speed
    FROM   PC pc JOIN Product p ON pc.model = p.model
    WHERE  ABS(pc.price - :desiredPrice) = (
               SELECT MIN(ABS(price - :desiredPrice)) FROM PC
           );

printf("Maker: %s  Model: %s  Speed: %.2f\n", maker, model, speed);
```
ALTERNATIVELY using Python3 with psycopg2
```python3
import psycopg2

# Connect to PostgreSQL
conn = psycopg2.connect(
    dbname="postgres",
    user="postgres",
    password="0000", #Only an example - never hardcode your password anywhere.
    host="localhost",
    port=5432
)
cursor = conn.cursor()

# Get desired price from user
desired_price = float(input("Enter desired price: "))

# Execute the query with parameterized values
query = """
    SELECT p.maker, pc.model, pc.speed
    FROM PC pc
    JOIN Product p ON pc.model = p.model
    WHERE ABS(pc.price - %s) = (
        SELECT MIN(ABS(price - %s)) FROM PC
    )
"""
cursor.execute(query, (desired_price, desired_price))

# Fetch the result
result = cursor.fetchone()

if result:
    maker, model, speed = result
    print(f"Maker: {maker}  Model: {model}  Speed: {speed:.2f}")
else:
    print("No matching PC found.")

# Close the cursor and connection
cursor.close()
conn.close()
```
---

### b) Find all laptops meeting minimum specs

A cursor is needed because multiple rows can be returned. Loop until `SQLSTATE` equals `"02000"` (not found).

```c
EXEC SQL BEGIN DECLARE SECTION;
    double minSpeed, minRam, minHd, minScreen;
    char   maker[30], model[10];
    double speed, ram, hd, screen, price;
EXEC SQL END DECLARE SECTION;

printf("Enter min speed, RAM, HD, screen: ");
scanf("%lf %lf %lf %lf", &minSpeed, &minRam, &minHd, &minScreen);

EXEC SQL DECLARE laptopCur CURSOR FOR
    SELECT p.maker, l.model, l.speed, l.ram, l.hd, l.screen, l.price
    FROM   Laptop l JOIN Product p ON l.model = p.model
    WHERE  l.speed  >= :minSpeed
      AND  l.ram    >= :minRam
      AND  l.hd     >= :minHd
      AND  l.screen >= :minScreen;

EXEC SQL OPEN laptopCur;
while (1) {
    EXEC SQL FETCH laptopCur
             INTO :maker, :model, :speed, :ram, :hd, :screen, :price;
    if (strcmp(SQLSTATE, "02000") == 0) break;
    printf("Maker: %s  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Screen: %.1f  Price: %.2f\n",
           maker, model, speed, ram, hd, screen, price);
}
EXEC SQL CLOSE laptopCur;
```
ALTERNATIVELY using Python3
```python3
import psycopg2

# Connect to PostgreSQL
conn = psycopg2.connect(
    dbname="postgres",
    user="postgres",
    password="0000", # Don't save your actual password anywhere!
    host="localhost",
    port=5432
)
cursor = conn.cursor()

# Get minimum specs from user
min_speed, min_ram, min_hd, min_screen = map(
    float, input("Enter min speed, RAM, HD, screen: ").split()
)

# Declare and execute the query
query = """
    SELECT p.maker, l.model, l.speed, l.ram, l.hd, l.screen, l.price
    FROM Laptop l
    JOIN Product p ON l.model = p.model
    WHERE l.speed  >= %s
      AND l.ram    >= %s
      AND l.hd     >= %s
      AND l.screen >= %s
"""
cursor.execute(query, (min_speed, min_ram, min_hd, min_screen))

# Fetch and print rows one by one (like cursor loop)
for row in cursor:
    maker, model, speed, ram, hd, screen, price = row
    print(
        f"Maker: {maker}  Model: {model}  Speed: {speed:.2f}  "
        f"RAM: {ram:.0f}  HD: {hd:.0f}  Screen: {screen:.1f}  Price: {price:.2f}"
    )

# Close the cursor and connection
cursor.close()
conn.close()
```

---

### c) Print all product specs for a given manufacturer

Query each product-type table with its own cursor. Only rows that exist for the given maker are printed.

```c
EXEC SQL BEGIN DECLARE SECTION;
    char manufacturer[30];
    char pcModel[10];  double pcSpeed, pcRam, pcHd, pcPrice;
    char lapModel[10]; double lapSpeed, lapRam, lapHd, lapScreen, lapPrice;
    char prModel[10], prType[10]; int prColor; double prPrice;
EXEC SQL END DECLARE SECTION;

printf("Enter manufacturer: ");
scanf("%s", manufacturer);

/* --- PCs --- */
EXEC SQL DECLARE pcCur CURSOR FOR
    SELECT pc.model, pc.speed, pc.ram, pc.hd, pc.price
    FROM   PC pc JOIN Product p ON pc.model = p.model
    WHERE  p.maker = :manufacturer;

EXEC SQL OPEN pcCur;
printf("PCs:\n");
while (1) {
    EXEC SQL FETCH pcCur INTO :pcModel, :pcSpeed, :pcRam, :pcHd, :pcPrice;
    if (strcmp(SQLSTATE, "02000") == 0) break;
    printf("  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Price: %.2f\n",
           pcModel, pcSpeed, pcRam, pcHd, pcPrice);
}
EXEC SQL CLOSE pcCur;

/* --- Laptops --- */
EXEC SQL DECLARE lapCur CURSOR FOR
    SELECT l.model, l.speed, l.ram, l.hd, l.screen, l.price
    FROM   Laptop l JOIN Product p ON l.model = p.model
    WHERE  p.maker = :manufacturer;

EXEC SQL OPEN lapCur;
printf("Laptops:\n");
while (1) {
    EXEC SQL FETCH lapCur
             INTO :lapModel, :lapSpeed, :lapRam, :lapHd, :lapScreen, :lapPrice;
    if (strcmp(SQLSTATE, "02000") == 0) break;
    printf("  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Screen: %.1f  Price: %.2f\n",
           lapModel, lapSpeed, lapRam, lapHd, lapScreen, lapPrice);
}
EXEC SQL CLOSE lapCur;

/* --- Printers --- */
EXEC SQL DECLARE prCur CURSOR FOR
    SELECT pr.model, pr.color, pr.type, pr.price
    FROM   Printer pr JOIN Product p ON pr.model = p.model
    WHERE  p.maker = :manufacturer;

EXEC SQL OPEN prCur;
printf("Printers:\n");
while (1) {
    EXEC SQL FETCH prCur INTO :prModel, :prColor, :prType, :prPrice;
    if (strcmp(SQLSTATE, "02000") == 0) break;
    printf("  Model: %s  Color: %s  Type: %s  Price: %.2f\n",
           prModel, prColor ? "yes" : "no", prType, prPrice);
}
EXEC SQL CLOSE prCur;
```
ALTERNATIVELY using Python3
```python3
import psycopg2

# Connect to PostgreSQL
conn = psycopg2.connect(
    dbname="postgres",
    user="postgres",
    password="0000",
    host="localhost",
    port=5432
)
cursor = conn.cursor()

# Get manufacturer from input
manufacturer = input("Enter manufacturer: ")

# PCs
print("PCs:")
cursor.execute("""
    SELECT pc.model, pc.speed, pc.ram, pc.hd, pc.price
    FROM PC pc
    JOIN Product p ON pc.model = p.model
    WHERE p.maker = %s
""", (manufacturer,))

for pcModel, pcSpeed, pcRam, pcHd, pcPrice in cursor:
    print(f"  Model: {pcModel}  Speed: {pcSpeed:.2f}  RAM: {pcRam:.0f}  "
          f"HD: {pcHd:.0f}  Price: {pcPrice:.2f}")

#LAptops
print("Laptops:")
cursor.execute("""
    SELECT l.model, l.speed, l.ram, l.hd, l.screen, l.price
    FROM Laptop l
    JOIN Product p ON l.model = p.model
    WHERE p.maker = %s
""", (manufacturer,))

for lapModel, lapSpeed, lapRam, lapHd, lapScreen, lapPrice in cursor:
    print(f"  Model: {lapModel}  Speed: {lapSpeed:.2f}  RAM: {lapRam:.0f}  "
          f"HD: {lapHd:.0f}  Screen: {lapScreen:.1f}  Price: {lapPrice:.2f}")

# Printers
print("Printers:")
cursor.execute("""
    SELECT pr.model, pr.color, pr.type, pr.price
    FROM Printer pr
    JOIN Product p ON pr.model = p.model
    WHERE p.maker = %s
""", (manufacturer,))

for prModel, prColor, prType, prPrice in cursor:
    color_str = "yes" if prColor else "no"
    print(f"  Model: {prModel}  Color: {color_str}  Type: {prType}  Price: {prPrice:.2f}")

# Close the cursor and connection
cursor.close()
conn.close()
```
---

### d) Find cheapest PC + printer within budget; prefer color printer

Try the cheapest combination with a **color** printer first. If no such combination exists within the budget at the required speed, fall back to any printer. `ORDER BY total_price ASC LIMIT 1` picks the cheapest valid pair.

```c
EXEC SQL BEGIN DECLARE SECTION;
    double budget, minSpeed;
    char   pcModel[10], prModel[10];
    int    found;
EXEC SQL END DECLARE SECTION;

printf("Enter budget and minimum PC speed: ");
scanf("%lf %lf", &budget, &minSpeed);

/* Try color printer first */
EXEC SQL
    SELECT pc.model, pr.model
    INTO   :pcModel, :prModel
    FROM   PC pc, Printer pr
    WHERE  pc.speed >= :minSpeed
      AND  pc.price + pr.price <= :budget
      AND  pr.color = TRUE
    ORDER BY pc.price + pr.price ASC
    LIMIT 1;

found = (strcmp(SQLSTATE, "02000") != 0);

/* Fall back to any printer if no color combo found */
if (!found) {
    EXEC SQL
        SELECT pc.model, pr.model
        INTO   :pcModel, :prModel
        FROM   PC pc, Printer pr
        WHERE  pc.speed >= :minSpeed
          AND  pc.price + pr.price <= :budget
        ORDER BY pc.price + pr.price ASC
        LIMIT 1;
    found = (strcmp(SQLSTATE, "02000") != 0);
}

if (found)
    printf("PC model: %s  Printer model: %s\n", pcModel, prModel);
else
    printf("No system found within budget.\n");
```

---

### e) Insert a new PC after checking for a duplicate model number

Count existing rows in `Product` with the given model. If none, insert into both `Product` and `PC`; otherwise print a warning.

```c
EXEC SQL BEGIN DECLARE SECTION;
    char   maker[30], model[10];
    double speed, ram, hd, price;
    int    cnt;
EXEC SQL END DECLARE SECTION;

printf("Enter maker, model, speed, RAM, HD, price: ");
scanf("%s %s %lf %lf %lf %lf", maker, model, &speed, &ram, &hd, &price);

EXEC SQL SELECT COUNT(*) INTO :cnt
         FROM Product WHERE model = :model;

if (cnt > 0) {
    printf("WARNING: model %s already exists.\n", model);
} else {
    EXEC SQL INSERT INTO Product(maker, model, type)
             VALUES (:maker, :model, 'pc');
    EXEC SQL INSERT INTO PC(model, speed, ram, hd, price)
             VALUES (:model, :speed, :ram, :hd, :price);
    printf("PC model %s inserted successfully.\n", model);
}
```
ALTERNATIVELY using Python3
```python3
import psycopg2

# Connect to PostgreSQL
conn = psycopg2.connect(
    dbname="your_dbname",
    user="your_username",
    password="your_password",
    host="localhost",
    port=5432
)
cursor = conn.cursor()

# Get user input
maker, model, speed, ram, hd, price = input(
    "Enter maker, model, speed, RAM, HD, price: "
).split()

# Convert numeric values
speed = float(speed)
ram = float(ram)
hd = float(hd)
price = float(price)

# Check if model already exists
cursor.execute("""
    SELECT COUNT(*) FROM Product WHERE model = %s
""", (model,))
cnt = cursor.fetchone()[0]

if cnt > 0:
    print(f"WARNING: model {model} already exists.")
else:
    # Insert into Product table
    cursor.execute("""
        INSERT INTO Product (maker, model, type)
        VALUES (%s, %s, 'pc')
    """, (maker, model))

    # Insert into PC table
    cursor.execute("""
        INSERT INTO PC (model, speed, ram, hd, price)
        VALUES (%s, %s, %s, %s, %s)
    """, (model, speed, ram, hd, price))

    # Commit the transaction
    conn.commit()
    print(f"PC model {model} inserted successfully.")

# Close cursor and connection
cursor.close()
conn.close()
```
---

## Exercise 9.6.1
### DISCLAIMER: As my Java is a little rusty, and this exercise is optional, I have used Gen AI to rewrite my solution from 9.3.1. into JDBC

**JDBC** (Java Database Connectivity) replaces embedded SQL with explicit Java objects: `Connection` for the session, `PreparedStatement` for parameterised queries (use `?` placeholders and `setXxx()` to bind values), and `ResultSet` for row iteration. Use `executeQuery()` for `SELECT` and `executeUpdate()` for `INSERT`/`UPDATE`/`DELETE`. Use **try-with-resources** to ensure statements and connections are closed automatically.

Each class declares the three connection constants at the top:

```java
static final String URL  = "jdbc:mysql://localhost:3306/mydb";
static final String USER = "dbuser";
static final String PASS = "dbpassword";
```

---

### a) Find the PC whose price is closest to the user's desired price

```java
import java.sql.*;
import java.util.Scanner;

public class ClosestPC {
    static final String URL  = "jdbc:mysql://localhost:3306/mydb";
    static final String USER = "dbuser";
    static final String PASS = "dbpassword";

    public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter desired price: ");
        double desiredPrice = sc.nextDouble();

        try (Connection conn = DriverManager.getConnection(URL, USER, PASS)) {
            String sql =
                "SELECT p.maker, pc.model, pc.speed " +
                "FROM PC pc JOIN Product p ON pc.model = p.model " +
                "WHERE ABS(pc.price - ?) = (SELECT MIN(ABS(price - ?)) FROM PC)";

            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setDouble(1, desiredPrice);
                stmt.setDouble(2, desiredPrice);
                ResultSet rs = stmt.executeQuery();
                if (rs.next())
                    System.out.printf("Maker: %s  Model: %s  Speed: %.2f%n",
                        rs.getString("maker"), rs.getString("model"), rs.getDouble("speed"));
            }
        }
    }
}
```

---

### b) Find all laptops meeting minimum specs

`ResultSet.next()` advances row by row; the `while` loop replaces the cursor fetch loop.

```java
import java.sql.*;
import java.util.Scanner;

public class LaptopSearch {
    static final String URL  = "jdbc:mysql://localhost:3306/mydb";
    static final String USER = "dbuser";
    static final String PASS = "dbpassword";

    public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter min speed, RAM, HD, screen: ");
        double minSpeed  = sc.nextDouble();
        double minRam    = sc.nextDouble();
        double minHd     = sc.nextDouble();
        double minScreen = sc.nextDouble();

        try (Connection conn = DriverManager.getConnection(URL, USER, PASS)) {
            String sql =
                "SELECT p.maker, l.model, l.speed, l.ram, l.hd, l.screen, l.price " +
                "FROM Laptop l JOIN Product p ON l.model = p.model " +
                "WHERE l.speed >= ? AND l.ram >= ? AND l.hd >= ? AND l.screen >= ?";

            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setDouble(1, minSpeed);
                stmt.setDouble(2, minRam);
                stmt.setDouble(3, minHd);
                stmt.setDouble(4, minScreen);
                ResultSet rs = stmt.executeQuery();
                while (rs.next())
                    System.out.printf(
                        "Maker: %s  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Screen: %.1f  Price: %.2f%n",
                        rs.getString("maker"), rs.getString("model"),
                        rs.getDouble("speed"), rs.getDouble("ram"),
                        rs.getDouble("hd"), rs.getDouble("screen"), rs.getDouble("price"));
            }
        }
    }
}
```

---

### c) Print all product specs for a given manufacturer

Run three separate queries—one per product type—reusing the same `Connection`.

```java
import java.sql.*;
import java.util.Scanner;

public class ManufacturerProducts {
    static final String URL  = "jdbc:mysql://localhost:3306/mydb";
    static final String USER = "dbuser";
    static final String PASS = "dbpassword";

    public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter manufacturer: ");
        String manufacturer = sc.next();

        try (Connection conn = DriverManager.getConnection(URL, USER, PASS)) {

            // PCs
            String pcSql =
                "SELECT pc.model, pc.speed, pc.ram, pc.hd, pc.price " +
                "FROM PC pc JOIN Product p ON pc.model = p.model WHERE p.maker = ?";
            try (PreparedStatement stmt = conn.prepareStatement(pcSql)) {
                stmt.setString(1, manufacturer);
                ResultSet rs = stmt.executeQuery();
                System.out.println("PCs:");
                while (rs.next())
                    System.out.printf("  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Price: %.2f%n",
                        rs.getString("model"), rs.getDouble("speed"),
                        rs.getDouble("ram"), rs.getDouble("hd"), rs.getDouble("price"));
            }

            // Laptops
            String lapSql =
                "SELECT l.model, l.speed, l.ram, l.hd, l.screen, l.price " +
                "FROM Laptop l JOIN Product p ON l.model = p.model WHERE p.maker = ?";
            try (PreparedStatement stmt = conn.prepareStatement(lapSql)) {
                stmt.setString(1, manufacturer);
                ResultSet rs = stmt.executeQuery();
                System.out.println("Laptops:");
                while (rs.next())
                    System.out.printf(
                        "  Model: %s  Speed: %.2f  RAM: %.0f  HD: %.0f  Screen: %.1f  Price: %.2f%n",
                        rs.getString("model"), rs.getDouble("speed"),
                        rs.getDouble("ram"), rs.getDouble("hd"),
                        rs.getDouble("screen"), rs.getDouble("price"));
            }

            // Printers
            String prSql =
                "SELECT pr.model, pr.color, pr.type, pr.price " +
                "FROM Printer pr JOIN Product p ON pr.model = p.model WHERE p.maker = ?";
            try (PreparedStatement stmt = conn.prepareStatement(prSql)) {
                stmt.setString(1, manufacturer);
                ResultSet rs = stmt.executeQuery();
                System.out.println("Printers:");
                while (rs.next())
                    System.out.printf("  Model: %s  Color: %s  Type: %s  Price: %.2f%n",
                        rs.getString("model"),
                        rs.getBoolean("color") ? "yes" : "no",
                        rs.getString("type"), rs.getDouble("price"));
            }
        }
    }
}
```

---

### d) Find cheapest PC + printer within budget; prefer color printer

Try the color-printer query first. If the `ResultSet` is empty, retry without the color filter.

```java
import java.sql.*;
import java.util.Scanner;

public class CheapestSystem {
    static final String URL  = "jdbc:mysql://localhost:3306/mydb";
    static final String USER = "dbuser";
    static final String PASS = "dbpassword";
    public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter budget and minimum PC speed: ");
        double budget   = sc.nextDouble();
        double minSpeed = sc.nextDouble();

        String baseSql =
            "SELECT pc.model AS pcModel, pr.model AS prModel " +
            "FROM PC pc, Printer pr " +
            "WHERE pc.speed >= ? AND pc.price + pr.price <= ? ";

        try (Connection conn = DriverManager.getConnection(URL, USER, PASS)) {
            String pcModel = null, prModel = null;

            // Try color printer first
            try (PreparedStatement stmt = conn.prepareStatement(
                    baseSql + "AND pr.color = TRUE ORDER BY pc.price + pr.price ASC LIMIT 1")) {
                stmt.setDouble(1, minSpeed);
                stmt.setDouble(2, budget);
                ResultSet rs = stmt.executeQuery();
                if (rs.next()) {
                    pcModel = rs.getString("pcModel");
                    prModel = rs.getString("prModel");
                }
            }

            // Fall back to any printer
            if (pcModel == null) {
                try (PreparedStatement stmt = conn.prepareStatement(
                        baseSql + "ORDER BY pc.price + pr.price ASC LIMIT 1")) {
                    stmt.setDouble(1, minSpeed);
                    stmt.setDouble(2, budget);
                    ResultSet rs = stmt.executeQuery();
                    if (rs.next()) {
                        pcModel = rs.getString("pcModel");
                        prModel = rs.getString("prModel");
                    }
                }
            }

            if (pcModel != null)
                System.out.printf("PC model: %s  Printer model: %s%n", pcModel, prModel);
            else
                System.out.println("No system found within budget.");
        }
    }
}
```

---

### e) Insert a new PC after checking for a duplicate model number

Use `executeUpdate()` for the `INSERT` statements. Both inserts share the same transaction; wrapping them in a try-with-resources block keeps the code clean.

```java
import java.sql.*;
import java.util.Scanner;

public class InsertPC {
    static final String URL  = "jdbc:mysql://localhost:3306/mydb";
    static final String USER = "dbuser";
    static final String PASS = "dbpassword";
    public static void main(String[] args) throws Exception {
        Scanner sc = new Scanner(System.in);
        System.out.print("Enter maker, model, speed, RAM, HD, price: ");
        String maker  = sc.next();
        String model  = sc.next();
        double speed  = sc.nextDouble();
        double ram    = sc.nextDouble();
        double hd     = sc.nextDouble();
        double price  = sc.nextDouble();

        try (Connection conn = DriverManager.getConnection(URL, USER, PASS)) {

            // Check for duplicate model number
            try (PreparedStatement check = conn.prepareStatement(
                    "SELECT COUNT(*) FROM Product WHERE model = ?")) {
                check.setString(1, model);
                ResultSet rs = check.executeQuery();
                rs.next();
                if (rs.getInt(1) > 0) {
                    System.out.println("WARNING: model " + model + " already exists.");
                    return;
                }
            }

            // Insert into Product
            try (PreparedStatement ins = conn.prepareStatement(
                    "INSERT INTO Product(maker, model, type) VALUES (?, ?, 'pc')")) {
                ins.setString(1, maker);
                ins.setString(2, model);
                ins.executeUpdate();
            }

            // Insert into PC
            try (PreparedStatement ins = conn.prepareStatement(
                    "INSERT INTO PC(model, speed, ram, hd, price) VALUES (?, ?, ?, ?, ?)")) {
                ins.setString(1, model);
                ins.setDouble(2, speed);
                ins.setDouble(3, ram);
                ins.setDouble(4, hd);
                ins.setDouble(5, price);
                ins.executeUpdate();
            }

            System.out.println("PC model " + model + " inserted successfully.");
        }
    }
}
```
