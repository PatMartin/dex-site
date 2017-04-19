+++
date = "2016-12-28T13:37:53-05:00"
title = "Dex Manual"

[menu.main]
  identifier = "docs"
+++
<style>
  html, body, .container, .container div {
    height : 98%;
    width: 100%;
    min-height: 95%;
    min-width: 100%;
  }
</style>

# TMI Manual

TMI stands for Table Manipulation Interpreter, or Too Much Information. Whichever you
please. TMI is a domain specific language written in ANTLR4 which allows for fast and
intuitive transformation of the data you have in to the data you want.

# Examples

## Selecting A Subset

```sql
select A, B;
```

## Selecting A Subset From A Named Table

```sql
select col1, col2 from t1;
```

## Selecting columns with difficult names

```sql
select "Server Name", "Server IP Address", "Server Port" from servers;
```

## Renaming Columns

```sql
select A as COL1, B as COL2;
```

## Copying a Column

This will preserve A and B and copy B into a new column called BCOPY from table T1.

```sql
select A, B, B as BCOPY from T1;
```

## Storing a query in a temporary result

```sql
myTable = select A, B from T1;
```
  
## Splitting a column by a regular expression

Suppose we have some data in a table called SERVERINFO such as:

| SERVER | PORTS |
| --- | --- |
| server1 | 80,8080 |
| server2 |21,22 |

We wish to split the PORTS column into one port per column. In TMI we simply:

```sql
split PORTS AS PORT in SERVERINFO with /,/;
```

Will return:

| SERVER | PORT |
|---|---|
|server1 | 80 |
|server1 | 8080 |
|server2 | 21 |
|server2 | 22 |

And if we wish to rename PORTS to the more logical name PORT at the same time:

## Splitting multiple columns by a regular expression

Suppose we have some data in a table called SERVERINFO such as:

| SERVERS | PORTS |
| --- | --- |
| server1,server2 | 80,8080 |
| server3,server4 |21,22 |

We wish to split the SERVERS and PORTS column into one server and one port per column. In TMI we simply:

```sql
split SERVERS as SERVER, PORTS as PORT in SERVERINFO with /,/;
```

Will return:

| SERVER | PORT |
|--------|------|
| server1 | 80 |
| server1 | 8080 |
| server2 | 80 |
| server2 | 8080 |
| server3 | 21 |
| server3 | 22 |
| server4 | 21 |
| server4 | 22 |

## Joining Two Tables

Suppose we have some data in a table called CUSTOMERS such as:

| CUSTOMER_ID | FIRST | LAST | ADDRESS | CITY |
|-------------|-------|------|---------|------|
| 1 | Patrick | Martin | 222 Dex Drive | Jacksonville |
| 2 | Tom | Fiacco | 234 Coaches Lane | Atlanta |
| 3 | Nancy | Drew | 123 Mystery Avenue | Charleston |

and another table containing customer orders called ORDERS:

| ORDER_ID	| CUSTOMER_ID | ITEM | PRICE |
|-----------|-------------|------|-------|
| 1 | 1	| HAMMER	| 12.95 |
| 2 | 2	| WRESTLING MAT	| 2600.00 |
| 3 | 3	| MAGNIFYING GLASS	| 10.59 |
| 4 | 1	| WRENCH	| 15.00 |
| 5 | 1	| DOOR	| 129.99 |
| 6 | 1	| HINGES	| 4.05 |
| 7 | 2	| WHISTLE	| 8.88 |

### Brute Force

We could join these into one big table via a brute force approach:

```sql
bruteForceTable = join CUSTOMERS, ORDERS;
```

Which gives us the product of the two:

| CUSTOMERS.CUSTOMER_ID | CUSTOMERS.LAST | CUSTOMERS.FIRST | CUSTOMERS.ADDRESS | CUSTOMERS.CITY | ORDERS.ORDER_ID | ORDERS.CUSTOMER_ID | ORDERS.ITEM | ORDERS.PRICE |
|-----------------------|----------------|-----------------|-------------------|----------------|-----------------|--------------------|-------------|--------------|
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 1 | 1 | HAMMER | 12.95 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 2 | 2 | WRESTLING MAT | 2600.00 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 3 | 3 | MAGNIFYING GLASS | 10.59 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 4 | 1 | WRENCH | 15.00 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 5 | 1 | DOOR | 129.99 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 6 | 1 | HINGES | 4.05 | 
| 1 | Martin | Patrick | 222 Dex Drive | Burgerton | 7 | 2 | WHISTLE | 8.88 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 1 | 1 | HAMMER | 12.95 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 2 | 2 | WRESTLING MAT | 2600.00 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 3 | 3 | MAGNIFYING GLASS | 10.59 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 4 | 1 | WRENCH | 15.00 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 5 | 1 | DOOR | 129.99 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 6 | 1 | HINGES | 4.05 | 
| 2 | Fiacco | Tom | 234 Coaches Lane | Atlanta | 7 | 2 | WHISTLE | 8.88 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 1 | 1 | HAMMER | 12.95 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 2 | 2 | WRESTLING MAT | 2600.00 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 3 | 3 | MAGNIFYING GLASS | 10.59 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 4 | 1 | WRENCH | 15.00 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 5 | 1 | DOOR | 129.99 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 6 | 1 | HINGES | 4.05 | 
| 3 | Drew | Nancy | 123 Mystery Avenue | Whosville | 7 | 2 | WHISTLE | 8.88 |

The simple join operation is a brute force combination where every row within the
first table is combined with every row in teh second table.  Thus, the 3×5 table
joined with the 7×4 table resulted in a table of dimensions 21×9 which is the product
of all the rows times the sum of the columns.

Things can get quite large quickly.

Now we can simply select the rows of interest, for instance the first,
last name and price and item which that customer has ordered.

```sql
joinedTable =
  select CUSTOMERS.FIRST as First, CUSTOMERS.LAST as Last,
         ORDERS.ITEM as Item, ORDERS.PRICE as Price
  from bruteForceTable
  where CUSTOMERS.CUSTOMER_ID == ORDERS.CUSTOMER_ID;
```

Results in:

| First | Last | Item | Price |
|-------|------|------|-------|
| Patrick | Martin | HAMMER | 12.95 |
| Patrick | Martin | WRENCH | 15.00 |
| Patrick | Martin | DOOR | 129.99 |
| Patrick | Martin | HINGES | 4.05 |
| Tom | Fiacco | WRESTLING MAT | 2600.00 |
| Tom | Fiacco | WHISTLE | 8.88 |
| Nancy | Drew | MAGNIFYING GLASS | 10.59 |

### Surgical Join

We could have performed the join in the above example in a single, more efficient
way:

```sql
efficientlyJoinedTable =
  select CUSTOMERS.FIRST as First, CUSTOMERS.LAST as Last,
    ORDERS.ITEM as Item, ORDERS.PRICE as Price
  from CUSTOMERS, ORDERS
  where CUSTOMERS.CUSTOMER_ID == ORDERS.CUSTOMER_ID;
```