## Exercise 2.4.1 (a-e): Relational Algebra Queries

### a) What PC models have a speed of at least 3.00?

**Query:**
π<sub>model</sub>(σ<sub>speed≥3.00</sub>(PC))

The selection filters PCs with speed ≥ 3.00, then projects the model attribute.

---

### b) Which manufacturers make laptops with a hard disk of at least 100GB?

**Query:**
π<sub>maker</sub>(σ<sub>hd≥100</sub>(Laptop ⋈ Product))

Join Laptop with Product to get manufacturer info, filter by hard disk size, then project maker.

---

### c) Find the model number and price of all products (of any type) made by manufacturer B.

**Query:**
π<sub>model,price</sub>(σ<sub>maker='B'</sub>(Product) ⋈ PC) ∪ π<sub>model,price</sub>(σ<sub>maker='B'</sub>(Product) ⋈ Laptop) ∪ π<sub>model,price</sub>(σ<sub>maker='B'</sub>(Product) ⋈ Printer)

Filter products by manufacturer B, join with each product type table to get prices, then union the results.

---

### d) Find the model numbers of all color laser printers.

**Query:**
π<sub>model</sub>(σ<sub>color=true ∧ type='laser'</sub>(Printer))

Filter printers by both color and laser type attributes, then project model.

---

### e) Find those manufacturers that sell Laptops, but not PC's.

**Query:**
π<sub>maker</sub>(σ<sub>type='laptop'</sub>(Product)) − π<sub>maker</sub>(σ<sub>type='pc'</sub>(Product))

Get makers of laptops and subtract makers of PCs using set difference.

---

## Exercise 2.4.2 (a-e): Expression Trees

### a) Expression tree for query 2.4.1(a)

```
         π(model)
            |
         σ(speed≥3.00)
            |
           PC
```

---

### b) Expression tree for query 2.4.1(b)

```
       π(maker)
          |
       σ(hd≥100)
          |
         ⋈
        / \
   Laptop  Product
```

---

### c) Expression tree for query 2.4.1(c)

```
                    ∪
                 /  |  \
                /   |   \
               /    |    \
       π(model,  π(model,  π(model,
        price)    price)    price)
          |         |         |
          ⋈         ⋈         ⋈
         / \       / \       / \
        /   \     /   \     /   \
   σ(maker  PC  σ(maker Laptop σ(maker Printer
    ='B')         ='B')         ='B')
      |             |             |
   Product      Product       Product
```

---

### d) Expression tree for query 2.4.1(d)

```
         π(model)
            |
    σ(color=true ∧ type='laser')
            |
         Printer
```

---

### e) Expression tree for query 2.4.1(e)

```
              −
             / \
            /   \
           /     \
      π(maker)  π(maker)
          |         |
    σ(type=     σ(type=
     'laptop')   'pc')
          |         |
       Product   Product
```

---

## Exercise 5.1.1: Projection as Set vs. Bag

**Query:** π<sub>speed</sub>(PC)

### As a Set:
{1.42, 1.86, 2.00, 2.10, 2.20, 2.66, 2.80, 3.06, 3.20}

Sets eliminate duplicates, so each unique speed value appears once.

### As a Bag:
[2.66, 2.10, 1.42, 2.80, 3.20, 3.20, 2.20, 2.20, 2.00, 2.80, 1.86, 2.80, 3.06]

Bags allow duplicates, preserving all original speed values from the PC relation.

### Average value as a Set:
(1.42 + 1.86 + 2.00 + 2.10 + 2.20 + 2.66 + 2.80 + 3.06 + 3.20) / 9 = 21.30 / 9 = **2.37 GHz**

### Average value as a Bag:
(2.66 + 2.10 + 1.42 + 2.80 + 3.20 + 3.20 + 2.20 + 2.20 + 2.00 + 2.80 + 1.86 + 2.80 + 3.06) / 13 = 32.30 / 13 = **2.48 GHz**

The bag average is higher because faster speeds (2.80, 3.20, 2.20) appear multiple times.

---

## Exercise 5.1.3: Relation as a Set vs. Bag

### a) 

Data of bore as a set: 
|bore|
|---|
|15|
|14|
|16|
|18|

As a bag:
|bore|
|---|
|15|
|16|
|14|
|16|
|15|
|15|
|14|
|18|

### b) Writing an algebraic expression for bores of ships as a bag.

π<sub>bore</sub>(Ships ▷◁ Classes)

---

## Exercise 5.1.4: Associative Law for Union on Bags

**Law:** (R ∪ S) ∪ T = R ∪ (S ∪ T)

The associative law for union holds for bags because bag union simply combines all tuples from both relations, preserving duplicates.

For bags, the union operation counts the total occurrences of each tuple across all relations being unioned. Since addition is associative, it doesn't matter how we group the unions - the final count of each tuple will be the same.

**Example:** If tuple t appears 2 times in R, 3 times in S, and 1 time in T:
- (R ∪ S) ∪ T: tuple t appears (2 + 3) + 1 = 6 times
- R ∪ (S ∪ T): tuple t appears 2 + (3 + 1) = 6 times

The order of operations doesn't affect the final multiplicities.
