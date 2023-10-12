> ```SQL
> -- comments out anything past this to the end of the line
> -- except for `string literals` SQL is white space and case insensitive
> ```

# Creating and Dropping Tables
> Stored relations, which are called tables. These are the kind of relation we deal with ordinarily — a relation that exists in the database and that can be modified by changing its tuples, as well as queried

```SQL
-- create a new relation
CREATE TABLE TableName (
col_name1 coltype1,
col_name2 coltype2, ...
);

-- create a new column in a table
ALTER TABLE TableName ADD new_col new_col_type;

-- remove a column from a table
ALTER TABLE TableName DROP col_name1 ;

-- remove the entire table
DROP TABLE TableName;
```

## SQL Types
Numeric Types:  
* INT/INTEGER: Represents whole numbers.
* DECIMAL/NUMERIC: Precision fixed-point numbers.
* FLOAT/REAL: Approximate numerical values with floating decimals.

Dates/Times:   
* DATE: Represents a date (year, month, day).
  * ```'1948-05-14'```
* TIME: Represents time in hours, minutes, and seconds.
  * ```'15:00:02.5'``` -- milliseconds are optional 
* DATETIME/TIMESTAMP: Represents both date and time.

Character Strings:
* CHAR(n): Fixed-length character string of length n.
  * If given value not long enough, pads in the value
* VARCHAR(n): Variable-length character string with a maximum length of n.
  * Does not pad if less than maximum length

Boolean Type:
* BOOLEAN: Represents TRUE, FALSE, or NULL.

## Default Values
> Give a value that is used as the default aka the value that appears in a column if no other value is known
```SQL
-- create a default value in a new relation
CREATE TABLE TableName (
col_name1 coltype1 DEFAULT default_val,
col_name2 coltype2, ...
);

-- create a default value for new column in a table
ALTER TABLE TableName ADD new_col new_col_type DEFAULT default_val;
```

***

# Constraints
constraints are restrictions on allowable data in the data base (in addition to structure and type restrictions imposed by table definitions)  
* declared as part of SCHEMA
* enforced by the DBMS
* ✔️ Protects data integrity (catch errors) and tells DBMS about data -> better optimization

**Types**
> NOT NULL
> Key
> Referential Integrity (Foreign Key)
> General Assertion

## NOT NULL
col can never contain NULL as a value
* whenever a new row is added or an existing row is modified, the column with the NOT NULL constraint must be provided with a value
> `..., col TYPE NOT NULL, ...`

## Primary Keys vs Unique
> **KEY CONSTRAINT**
> Two tuples in R cannot agree on all of the attributes in key set, unless one of them is NULL. Any attempt to insert or update a tuple that violates this rule causes the DBMS to reject the action that caused the violation.

**Ways to add a Key**  
1. We may declare one attribute to be a key when that attribute is listed in the relation schema.
2. We may add to the list of items declared in the schema (which so far have only been attributes) an additional declaration that says a particular attribute or set of attributes forms the key
   If there is more than one attribute in a key set, have to do it with method 2  

**Types of Keys**  
*just chose which one to be your primary key, and which one to be your unique key*  
1. Primary Key: the one that is going to act as your key for your relation <- at most one per table
   * There is at most one primary key
   * We like primary keys because they are sorted by these values leading to faster speedup
2. Unique: not going to act as the key, just need to say so you get the key constraint <- secondary index
   * You can have multiple unique key sets  
   * These do not provide as much speedup  

```SQL
-- one primary key with one attribute, one unique key with one attribute
CREATE TABLE Users (
    UserID INT NOT NULL PRIMARY KEY,
    Username VARCHAR(50) NOT NULL UNIQUE,
    Email VARCHAR(100) UNIQUE
);

-- multi-attribute primary key
CREATE TABLE StudentCourses (
    StudentID INT NOT NULL,
    CourseID INT NOT NULL,
    PRIMARY KEY (StudentID, CourseID)
);

-- one primary key one attribute, and two unique keys with one attribute
CREATE TABLE Users (
    UserID INT NOT NULL PRIMARY KEY,
    Username VARCHAR(50) NOT NULL UNIQUE,
    Email VARCHAR(100) NOT NULL UNIQUE,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

-- one primary key two attribute, and two unique keys with two attribute
CREATE TABLE StudentCourses (
    StudentID INT NOT NULL,
    CourseCode VARCHAR(10) NOT NULL,
    StudentEmail VARCHAR(100) NOT NULL,
    CourseTitle VARCHAR(100) NOT NULL,
    StudentPhone VARCHAR(15) NOT NULL,
    CourseInstructor VARCHAR(50) NOT NULL,
    
    PRIMARY KEY (StudentID, CourseCode),
    UNIQUE (StudentEmail, CourseTitle),
    UNIQUE (StudentPhone, CourseInstructor)
);
```

## Referential Keys  
if R.a references S.a, and x appears in R then it must also appear in S (no dangling pointers)  
* Referencing (R) --> Referenced (S)
* Referenced column (S.a) must be a PRIMARY KEY
* Referecing columns form a FOREIGN KEY

> FOREIGN KEY can be declared explicitely with REFERENCES and also with just REFERENCING (if that attr is the only referenced attr in S)
> * FOREIGN KEYs can contain duplicates (by default many-one -- same as arrow)
> * however can also use as primary key in referencing table

```SQL
-- User has a primary key uid.
-- Group has a primary key gid.
-- Event has a composite primary key (eid, date).
-- The Member table will:
 -- Use uid as a foreign key referencing User.
 -- Use gid as both a foreign key referencing Group and part of its own primary key.
 -- Have a composite foreign key (eid, date) referencing the composite primary key in the Event table.

CREATE TABLE Member (
    uid INTEGER NOT NULL REFERENCES User(uid),
    gid CHAR(10) NOT NULL,
    eid INT NOT NULL,
    date DATE NOT NULL,
    PRIMARY KEY(uid, gid),
    FOREIGN KEY (gid) REFERENCES Group(gid),
    FOREIGN KEY (eid, date) REFERENCES Event(eid, date)
);
```

**Enforcing Referential Integrity**  
> happens on an insert/update to the referencing table  
> happens on a delete/update to the referenced table
> * this can be handled by rejecting (if it breaks a reference), cascade (update/delete all referring rows), set all references to NULL)


### Deferred Constraint Checking + Alter Schema After
**To Alter Schema After**   
Add Primary Key:   
> `ALTER TABLE [TableName] ADD CONSTRAINT PK_[TableName] PRIMARY KEY ([ColumnName]);`  

Add Foreign Key:  
> `ALTER TABLE [TableName]
ADD CONSTRAINT FK_[TableName]_[ReferencedTableName]
FOREIGN KEY ([Column1Name], [Column2Name]) REFERENCES [ReferencedTableName]([ReferencedColumn1Name],[ReferencedColumn2Name]);`


**Deferred Constraint Checking**
Checking constraints only at the end of a transaction

```SQL
-- no chicken or egg, can't insert non-atomically otherwise first insert always breaks
CREATE TABLE Dept
 (name CHAR(20) NOT NULL PRIMARY KEY,
 chair CHAR(30) NOT NULL REFERENCES Prof(name));
CREATE TABLE Prof
 (name CHAR(30) NOT NULL PRIMARY KEY,
 dept CHAR(20) NOT NULL REFERENCES Dept(name));
```

## General Assertion and Checks
### General Assertion <- More Hypothetical
> `CREATE ASSERTION assertion_name CHECK assertion_condition;`
> * assertion_condition checked for ALL modifications that could potentially violate <- lots of checking so not really supported

### Checks <- less constrained way
> on a single col: `...col TYPE CHECK(condition)`  
> can also be done at end and reference 1/mult col: `..., CHECK(condition),...`  
> associated with a SINGLE TABLE and only checked when tuple/attribute (of that table) is inserted/updated  
> * true AND UNKNOWN are fine (only FALSE rejected)  
> * notice it does not check if other table referenced are used -- so don't use with other tables  

```SQL
-- good check
CREATE TABLE User(...
 age INTEGER CHECK(age IS NULL OR age > 0),
 ...);

-- bad check (doesn't catch if member changed)
CREATE TABLE Member
 (uid INTEGER NOT NULL,
 CHECK(uid IN (SELECT uid FROM User)),
 ...);
```

***

# Updating Table Rows
## INSERT
To insert one row do: `INSERT INTO Table VALUES (col1val, col2val,...);`    
To insert the result of a query do: `INSERT INTO Table (subquery);`  

## DELETE
To delete everything from a table do: `DELETE FROM Table;`
To delete any row that satisfies WHERE do: `DELETE FROM Table WHERE ...;`


## UPDATE
Update the col value in all rows do: `UPDATE Table SET coli = val;`  
To do it for specific rows add a where clause  
You can SET to a subquery or compute WHERE using a subquery  
*If you are using a subquery in update, the subqueries are calculated over the OLD table*

***

# Select From Where
> SELECT specifies the columns you want to retrieve.  
> FROM specifies the table(s) you want to retrieve.  
> *These two parts are REQUIRED for any quere.*  
> The third (optional) statement is WHERE which filters row results based on a condition.
> * In queries must ALWAYS be written in this order, but when logicing about them think in order of FROM, WHERE, SELECT
>  
> Equivalent to Relational Algebra: $\pi_{A1, A2,..., An} (\sigma_{condition}(R1 \times R2 \times ...\times Rm))$

```SQL
SELECT A1, A2, ... An  
FROM R1, R2, ... Rm
WHERE condition

-- to evalute
-- For each t1 in R1: ...
--   For each tm in Rm:
--     If condition is true over t1, .. tm:
--       Output A1, ... An as a row (and do any computations for these if necessary)
```
#### Fun Stuff About Select 
* `SELECT *` means select all columns
* SELECT can contain expressions
  * `SELECT 2023-age` will return the all values of column age subtracted from 20
* you can also select constant values not assicated with any table it just return a result set with a single row containing that constant value
 * use when when checking for the existence of rows or when the exact value returned isn't crucial, but the presence or absence of data is
 * use to create a default value for join
 ```SQL
 CREATE VIEW GradeCount(teacher_id, grade, num) AS
  WITH GradeRecord(id, grade) AS
   (SELECT teacher_id, grade
   FROM Offering, Grade
   WHERE Offering.subject_name = Grade.subject_name
         AND Offering.year = Grade.year)
 (SELECT id, grade, COUNT(*) FROM GradeRecord GROUP BY id, grade)
 UNION ALL
 (SELECT id, grade, 0
  FROM (SELECT DISTINCT id FROM GradeRecord) t, -- all teachers (with least one grade record)
       (SELECT DISTINCT grade FROM Grade) g -- all possible grades
  WHERE NOT EXISTS
       (SELECT * FROM GradeRecord
       WHERE id = t.id AND grade = g.grade));
 -- rtn (teacher_id, grade, num) for every teacher_id, grade pair and set it equal to 0 if one of the groups do not exist
 ```
 
#### Confusing Names
If columns from two tables share same name use `table_name.column_name` to clear up ambiguity (can do this for unique columns too, but not required)  

#### Alias/Renaiming
Can rename columns and tables
```SQL
SELECT a1.col AS col1, a2.col col2
FROM Table AS a2, Table a2

-- AS is optional
```

#### Pattern Matching
> `s LIKE p` sees if string s matches pattern p. Patterns have special characters `%` and `_`.
> * `%` matches any number of characters
> * `_` matches exactly one character
```SQL
s LIKE 'a%'    --Finds any values that start with "a".
s LIKE '%a'    --Finds any values that end with "a".
s LIKE '%or%'  --Finds any values that have "or" in any position.
s LIKE '_r%'   --Finds any values that have "r" in the second position.
```

***

# Bag vs Set Semantics
> **Set**  
> No duplicates  
> Relational model and algebra use set semantics  
>   
> **Bag**  
> Duplicates allowed  
> * more efficient because don't need to eliminate duplicates
> * more useful because duplicate count has meaning (ex. distribution)    
> SQL uses bag semantics by default

## Bag -> Set with DISTINCT
```SQL
SELECT DISTINCT ...
FROM ...
WHERE ...

-- For each t1 in R1: ...
--   For each tm in Rm:
--     If condition is true over t1, .. tm:
--       Output A1, ... An as a row (and do any computations for these if necessary)
-- If DISTINCT is present, eliminate duplicate rows in output
```

## Set Operations
> Since `UNION, EXCEPT, INTERSECT` correspond directly to $\cup \setminus \cap$ from set theory. These are evaluated using set semantics
> * Duplicates in input tables, if any, are first eliminated
> *  Duplicates in result are also eliminated (for UNION)

> `UNION ALL, EXCEPT ALL, INTERSECT ALL` is their bag semantic equation. Think of these as **some sort of count**
> Let count stand for how many times a row appears in a table
> `UNION ALL`: sum up the counts from two tables  
> `EXCEPT ALL`: proper-subtract the two counts  
> `INTERCECT ALL`: take the minimum of the two counts  

```SQL
-- Poke (uid1, uid2, timestamp) where uid1 is the person who pokes and uid 2 is the person who gets poked

(SELECT uid1 FROM Poke)  -- select all unique people who poked
EXCEPT
(SELECT uid2 FROM Poke); -- select all unique people who got poked
-- Users who poked others but never got poked by others

(SELECT uid1 FROM Poke)  -- select all pokers
EXCEPT ALL               -- as long as you were poking more then you were poked you'll appear
(SELECT uid2 FROM Poke); -- select all people who poked
-- Users who poked others more than others poke them
```

***

# Subqueries
> Temporary tables, which are constructed by the SQL language processor when it performs its job of executing queries and data modifications. These relations are then thrown away and not stored.

## Table Subqueries 
> when the result of a subquery is a table, you can use it anywhere you would use a table (ex. btw set operations, in FROM clause...)

```SQL
-- in FROM, then you can use alias like an other table
SELECT ... 
FROM (SELECT ... FROM ... Where ...) AS alias
WHERE ...

-- in SET
(SELECT ... FROM ... Where ...)
UNION (ALL) | EXCEPT (ALL) | INTERCECT (ALL) 
(SELECT ... FROM ... Where ...) 

-- in JOIN
(SELECT ... FROM ... Where ...)
JOIN
(SELECT ... FROM ... Where ...) AS alias
ON ...

-- by giving table subqueries an alias, makes it easier to conceptualize
```

## Scalar Subqueries
> when the result of a subquery is a scalar, you can use it anywhere you can use a scalar (such as part of a computation in SELECT, something in WHERE, HAVING...)  
>   
> **BE CAREFUL THOUGH**  
> * if you aren't SELECT a key, no guarantee that it will only one row -- if returns more than one row you get runtime error
> * even if you SELECT a key, there is a chance that there will be no rows. In that case it returns `NULL` and depending on the query, can break your query

```SQL
SELECT salesperson, amount
FROM sales
WHERE amount > (SELECT AVG(amount) FROM sales);
```

## Correlated Subquery
Subquery that references tuple variables in surrounding queries    
* Because it is correlated, you have to evaluate for every new assignment of the surrounding query instead of just once

> Scoping rule (aka which table a column belongs to    
> * start with immediate query   
> * if not found, look in the one surrounding that; repeat if necessary   
*use table_name.column_name and AS to avoid confusion*   

## IN subqueries
> `x IN (subquery)` checks if x is in the result of subquery  
> s true if and only if s is equal to one of the values in R. Likewise, s NOT IN R is true if and only if s is equal to no value in R.    

## EXISTS subqueries
> `EXITS (subquery)` checks if the result of subquery is not empty

## Quantified subqueries (for all, exists)
### Universal quantification (for all):  
> `... WHERE x op ALL(subquery) ...` is true iff for all t in the result of subquery, x op t  

### Existential quantification (exists)
> `... WHERE x op ANY||SOME(subquery) ...` is true iff there exists some t in subquery result such that x op t  
> * note ANY and SOME both mean exists  

## Negation
add a NOT  
> `... WHERE NOT x op ANY||SOME||ALL(subquery) ...`  
> `x NOT IN||EXISTS (subquery)`  

> Note that when comparing a tuple with members of a relation R, we must compare components using the assumed standard order for the attributes of R.  

***
# Aggregates
`Count, Sum, AVG, MIN, MAX( [DISTINCT] col)` or `COUNT(*)` which counts the number of rows   
* include DISTINCT if you want set (no dups)
1. SUM produces the sum of a column with numerical values.  
2. AVG produces the average of a column with numerical values.   
3. MIN and MAX, applied to a column with numerical values, produces the smallest or largest value, respectively. When applied to a column with character-string values, they produce the lexicographically (alphabetically) first or last value, respectively  
4. COUNT produces the number of (not necessarily distinct) values in a column. Equivalently, COUNT applied to any attribute of a relation produces the number of tuples of that relation, including duplicates.  

## Grouping
> `SELECT ... FROM ... WHERE ... GROUP BY list_of_columns;` First breaks R into groups with each group containing all tuples who have the same value in all the grouping attributes, if no grouping attributes entire R is one group. Then, for each group calculate the aggregate value  
> number of groups = number of rows in the final output  

### Restriction on Aggregated/Group By
Every column in SELECT must either be  
* aggregated  
* GROUP BY col

Otherwise, SELECT expresion would produce MORE than ONE value per group and how would you represent that  

## Having
> `SELECT ... FROM ... WHERE ... GROUP BY ... HAVING condition;` filters group based on GROUP properties (aggregate values or GROUP BY col values)

* An aggregation in a HAVING clause applies only to the tuples of the group being tested.  
* Any attribute of relations in the FROM clause may be aggregated in the HAVING clause, but only those attributes that are in the GROUP BY list may appear unaggregated in the HAVING clause (the same rule as for the SELECT clause).  

**Semantics**  
* Compute FROM  
* Compute WHERE   
* Computer GROUP BY (group rows according to same values of GROUP BY columns)  
* Compute HAVING (basically selection over groups)  
* Compute SELECT for each group  
 * For aggregation functions wtih DISTINCT, first elimate dups within the groups   

*** 

# Ordering  
> `SELECT [DISTINCT] …FROM … WHERE … GROUP BY … HAVING … ORDER BY output_column [ASC|DESC], …;` sort output by values in
> * if multiple order, sort by first first then sort the ones that were same in regards to the first sort by the second  
> * instead of using column for sort, can also do it by their sequence number (where they appear in the SELECT)   
> * STRICTLY SPEAKING, only OUTPUT columns can appear in ORDER BY clause   
> * The order is by default ascending, but we can get the output highest-first by appending the keyword DESC (for “descending”) to an attribute.   
> * not actuallly part of the aggregate stuff, could just do SELECT * FROM .. ORDER BY   

**Semantics**  
After SELECT list has been computed and optional duplicate elimination has been carried out, sort the output according to ORDER BY specification  

```SQL
-- list all users, sort by pop (desc) and name (asc)
SELECT uid, name, age, pop
FROM User
ORDER BY pop DESC, name;
-- or ORDER BY 4 DESC, 2;
```

***

# Common Table Expressions (WITH)
Defines temporary tables to be used by  
* Other tables defined IN the SAME WITH
* the actual_query

The final result is the actual_query ONLY, nothing in the WITH is saved/returned

```SQL
WITH table1(column11, column12, …) -- column names are optional
  AS (query_definition_1),
 table2(column21, column22, …)     -- additional temorary tables as needed
  AS (query_definition_2), …
actual_query;
```

## WITH Recursion
General Structure:
```SQL
WITH RECURSIVE Recursive(optional col names) AS (
    -- Base case: this selects the initial data to start the recursion
    SELECT [InitialColumns]
    FROM [TableName]
    WHERE [InitialCondition]

    UNION ALL

    -- Recursive case: this references the Recursive to continue the recursion
    SELECT [RecursiveColumns]
    FROM Recursive, ...                    -- some sort of join on RECURSE (could be using from, or actual JOIN)
    WHERE [AdditionalConditions, if any]
);
Query using relation defined in WITH clause
```

### Fixed Point q(T) = T
```
# Calculating the Unique Minimal Fixed Point if q is Monotone
Start with an empty table: T ← ∅
• Evaluate q over T
• If the result is identical to T, stop; T is a fixed point
• Otherwise, let T be the new result; repeat

-- q is query and T is table that is the fixed point
```
> the intuition is you start off knowing nothing  
> the base case means you deduce a starting fact  
> in subsequent recursive steps, you use facts deduced in previous steps to learn more facts  
> stop when no new facts can be proven

## Linear Recursion
Recursive Definition makes only one reference to itself
```SQL
-- Non-linear
WITH RECURSIVE Ancestor(anc, desc) AS
 ((SELECT parent, child FROM Parent)
 UNION
 (SELECT a1.anc, a2.desc
  FROM Ancestor a1, Ancestor a2
  WHERE a1.desc = a2.anc))

--Linear
 WITH RECURSIVE Ancestor(anc, desc) AS
 ((SELECT parent, child FROM Parent)
 UNION
 (SELECT anc, child
  FROM Ancestor, Parent
  WHERE desc = parent))
```
Linear recursion easier to implement  
* linear: you join newly generated rows with a static table
* nonlinear: you join newly generated rows with all existing rows

Non linear recursion may take fewer steps to converge but may preferm more work (end up with two different ways to derive the same thing)
* linear: does everything one step at a time  
* nonlinear: does multiple things -- lead to solution faster but also repetition

## Mutual Recursion
n temporary recursion tables that can all reference each other
```SQL
WITH RECURSIVE R1 AS Q1,...,
     RECURSIVE Rn AS Qn
Q;
-- Q and Q1,…,Qn may refer to R1,…,Rn
```

### Fixed Point for Mutual Recursion
```
1. R1 ← ∅,…,Rn ← ∅
2. Evaluate Q1,…,!n using the current contents of R1,...,Rn:
 R1new <- Q1,...,Rnnew <- Qn
3. If there exits i s.t. Rinew != Ri
 Ri <- Rinew and repeat 2
4. Otherwise, you found fixed point and can know calculate Q using R1,..Rn
```

## Fixed Point Nuances - Monotone/Non-monotone - Minimal
Fixed points are not unique   
💡 if q is MONOTONE, then all not unique fixed points must contain the fixed point we computed from fixed-point iteration starting with the empty set  
* thus the UNIQUE MINIMAL FIXED POINT is the right answer for monotone queries

> if q is non-monotone fixed points may not converge (flip flop btw numbers) or there could be multiple minimal fixed points

## Recursion with Non-monotone
### Dependency Graph
> * one node for each table defined in WITH
> * direct edge R->S if R is defined in terms of S
> * label directed edge '-' if query defining R is not monotone WITH RESPECT to S
>  * EXCEPT, NOT IN, ...

💡 legal is if there is NO CYCLE with '-' edge -- stratified negation

### Stratified Negation
Stratum of a node R is the maximum number of '-' edges on any path from R in the dependency graph  

Evaluation:  
> compute tables with lowest-stratum first
> for each stratum, use fixed-point iteration on all nodes in that stratum


***

# Unknown Values -> NULL
Unknown values arise when we don't have info on something (unknown) or if a value is not applicable

## NULL
NULL exists for every domain/type and SQL provides special rules for dealing with NULLs

### Computing
* NULL +/-/operation... NULL||other value = NULL
* When we compare a NULL value and any value, including another NULL, using a comparison operator like = or >, the result is UNKNOWN. The value UNKNOWN is another truth-value, like TRUE and FALSE
 * TRUE = 1, FALSE = 0, UNKNOWN = .5
 * x AND y = min(x, y) || x OR y = max(x,y) || NOT x = 1-x
 * WHERE and HAVING clauses only select rows if condition is TRUE (doesn't take UNKNOWN) 
* Aggregate functions ignore NULL, except COUNT(*) (since it counts rows)  
* Evaluating aggregation functions (except COUNT) on an empty collection returns NULL; converting a empty collection to a scalar also gives NULL  

### NULL Equivalence Breaking
```
SELECT AVG(pop) FROM User;
SELECT SUM(pop)/COUNT(*) FROM User;
-- AVG computed without NULL, COUNT(*) includes NULL

SELECT * FROM User;
SELECT * FROM User WHERE pop = pop;
-- First includes rows where pop is NULL, since pop = pop is UNKNOWN if pop is NULL, second removes
```

### NULL Predicates and Functions
#### IS [NOT] NULL
> `IS NULL and IS NOT NULL` allows you to check a value is NULL bc you canpt do = NULL bc that always unknown

#### Coalesce
> `COALESCE(arg1, arg2, ...)` rtn the first arg that is NOT NULL

#### NullIf
> `NULLIF(arg1, arg2)` rtn NULL if the arguments are EQUAL

***

# Joins
**Students**
| StudentID | StudentName |
|-----------|-------------|
| 1         | Alice       |
| 2         | Bob         |
| 3         | Charlie     |

**Courses**
| StudentID | CourseName  |
|-----------|-------------|
| 1         | Math        |
| 3         | History     |
| 4         | Chemistry   |

*One way to get cross product is by putting multiple relations in WHERE*  
`SELECT * FROM Students s, Courses c WHERE Students.StudentID = Courses.StudentID`

| s.StudentID | StudentName | c.StudentID | CourseName  |
|-------------|-------------|-------------|-------------|
| 1           | Alice       | 1           | Math        |
| 3           | Charlie     | 3           | History     |

## Outerjoin
*to depict its the little bowtie for natural join, but wherever you are doing dangling rows add little pointy lines out*  

### Full Outerjoin
includes all rows in the result of R NATURAL JOIN S and   
* dangling R rows (those that do not join with any S rows) padded with NULL's for S's columns
* dangling S rows padded with NULL's for R's columns

> `(subquery/table) FULL OUTER JOIN (subquery/table) ON condition;`  

| StudentID | StudentName | CourseName |
|-----------|-------------|------------|
| 1         | Alice       | Math       |
| 2         | Bob         | NULL       |
| 3         | Charlie     | History    |
| 4         | NULL        | Chemistry  |


### Left Outerjoin
includes all rows in the result of R NATURAL JOIN S and dangling R rows padded with NULLS  
> `(subquery/table) LEFT OUTER JOIN (subquery/table) ON condition;`  

| StudentID | StudentName | CourseName |
|-----------|-------------|------------|
| 1         | Alice       | Math       |
| 3         | Charlie     | History    |
| 4         | NULL        | Chemistry  |

### Right Outerjoin
includes all rows in the result of R NATURAL JOIN S and dangling S rows padded with NULLS  
> `(subquery/table) RIGHT OUTER JOIN (subquery/table) ON condition;`

| StudentID | StudentName | CourseName |
|-----------|-------------|------------|
| 1         | Alice       | Math       |
| 3         | Charlie     | History    |
| 4         | NULL        | Chemistry  |


## Theta Joins (like from RA)
> `(subquery/table) JOIN (subquery/table) ON condition;`

| StudentID | StudentName | StudentID | CourseName |
|-----------|-------------|-----------|------------|
| 1         | Alice       | 3         | History    |
| 1         | Alice       | 4         | Chemistry  |
| 2         | Bob         | 3         | History    |
| 2         | Bob         | 4         | Chemistry  |
| 3         | Charlie     | 4         | Chemistry  |

## Natural Join
> `(subquery/table) NATURAL JOIN (subquery/table;`

| StudentID | StudentName | CourseName |
|-----------|-------------|------------|
| 1         | Alice       | Math       |
| 3         | Charlie     | History    |

***

# Views
> Views, which are relations defined by a computation. These relations are not stored, but are constructed, in whole or in part, when needed  
> * defined by a query which describes how to compute the view contents when needed (store the view definition query instead of contents)   
> * kinda like temporary tables defined by WITH but are visible to ALL STATEMENTS as they ARE PART OF THE SCHEMA

## Creation and Dropping and use
> Creation: `CREATE VIEW Name AS (query)`
> Drop: `DROP VIEW Name`
* tables used in defining a view (the ones that appear in any FROM used in the view) are called "base tables"

> to use, littery just put its name in the FROM clause (just like a normal table)
> to evaluate, replace reference to view by its definition and evaulate as normal  

## Modifying Views 
One way to change things in a view is the change the corresponding thing in the base table  
* However sometimes this is impossible
* Also sometimes multiple ways to do a change
```SQL
-- changing using change in base table
 CREATE VIEW UserPop AS
 SELECT uid, pop FROM User;
-- to do DELETE FROM UserPop WHERE uid = 123; do
DELETE FROM User WHERE uid = 123;

-- impossible situation
CREATE VIEW PopularUser AS
 SELECT uid, pop FROM User
 WHERE pop >= 0.8;
-- can't do INSERT INTO PopularUser VALUES(987, 0.3); bc no matter how you change User, it will never end up in PopularUser
```
> bc of the impossible case need to add WITH CHECK OPTION to remove wrong cases

### INSTEAD OF Trigger to Modify Views
```SQL
CREATE TRIGGER TrigerName
INSTEAD OF INSERT|DELETE|UPDATE ON ViewName  --for UPDATE can also do UPDATE [OF column] ON table
REFERENCING [TRANSITION VARIABLES] AS alias
FOR EACH ROW|STATEMENT
-- condition or no condition
UPDATE BaseTable SET
 [whatever update should be]
```
***

# Triggers
trigger is an event-condition-action rule (if even occurs, test condition; if condition is satisfied, execute action)  
```SQL
CREATE TRIGGER TriggerName
---event
[TIMING] [EVENT]
---trigger notation
REFERENCING [TRANSITION VARIABLES] AS alias
[GRANULARITY]
---condition (can remove if you want to do for on)
WHEN condition
---action
action;
```
#### [TIMING] - when action can be executed  
* AFTER triggering event  
 * Use Case: You need to perform some action after the data has been successfully modified in the table, typically in response to the change.  
    Example: Logging changes, or updating another table based on the change in the current table.   
* BEFORE triggering event  
 * BEFORE are used to "condition" data or raise an error in the trigger body to abort transaction if it is no bueno  
 * Use Case: You want to check or modify the data before it is written to the table.  
    Example: Enforcing a business rule, such as making sure a certain field is not NULL before inserting or updating.  
* INSTEAD OF triggering event on views  

#### [EVENT] - possible events  
* INSERT ON Table  
* DELETE ON Table  
* UPDATE [OF col] ON table  

#### [GRANULARITY] - when trigger can be activated  
* FOR EACH ROW modified    
 * Use Case: You want to execute the trigger's actions for each individual row affected by the triggering statement. This is useful when the action you want to take depends on the data in each specific row.      
   Example: Updating an audit log with the specific details of each row that was changed.     
* FOR EACH STATEMENT that performs modification -- rtn table with all modified rows  
 * Use Case: Take an action once per triggering SQL statement. This is useful for actions that don't depend on the specific data in each row or that depend on how many rows affected.  
    Example: If the number of users inserted by this statement exceeds 100 and their average age is below 13, then …  

> some triggers such as those that want to deal with ex how many rows satisfy condiiton are only possible at statement lvl   
> * however row lvl triggers easier to implement and use less state (for things that don't modify too many rows)    

#### [TRANSITION VARIABLES] - things SQL gives you based on event  
* OLD ROW: the modified row before the triggering event
* NEW ROW: the modified row after the triggering event
* OLD TABLE: a hypothetical read-only table containing all rows to be modified before the triggering event
* NEW TABLE: a hypothetical table containing all modified rows after the triggering event

> BEFORE/AFTER INSERT row/statement lvl triggers use only NEW ROW/TABLE  
> BEFORE/AFTER UPDATE row/statment lvl triggers use OLD and NEW ROW/TABLE
> BEFORE/AFTER DELETE row/statement lvl trigger use only OLD ROW/TABLE
> * row for row lvl, table for table lvl   

> Be careful for recursive firing of triggers (some leave to prog, other restrict number of actions, or set maximum lvl of recursion)

### Condition Checking
* check after a BEFORE trigger (bc in trigger, could potentially fix violation)  
* before an AFTER trigger (bc don't want to do event if constraint violate)  
 * also AFTER triggers can cause cascaded deltes based on referential integrity constraint violoation   



