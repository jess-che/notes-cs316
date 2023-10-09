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
# Keys
## Primary Keys vs Unique
> **KEY CONSTRAINT**
> Two tuples in R cannot agree on all of the attributes in key set, unless one of them is NULL. Any attempt to insert or update a tuple that violates this rule causes the DBMS to reject the action that caused the violation.

**Ways to add a Key**  
1. We may declare one attribute to be a key when that attribute is listed in the relation schema.
2. We may add to the list of items declared in the schema (which so far have only been attributes) an additional declaration that says a particular attribute or set of attributes forms the key
   If there is more than one attribute in a key set, have to do it with method 2  

**Types of Keys**  
*just chose which one to be your primary key, and which one to be your unique key*
1. Primary Key: the one that is going to act as your key for your relation
  * There is at most one primary key
  * We like primary keys because they are sorted by these values leading to faster speedup
3. Unique: not going to act as the key, just need to say so you get the key constraint
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
```
#### Fun Stuff About Select 
* `SELECT *` means select all columns
* SELECT can contain expressions
  * `SELECT 2023-age` will return the all values of column age subtracted from 20
 
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

# Subqueries
> Views, which are relations defined by a computation. These relations are not stored, but are constructed, in whole or in part, when needed

# Views
> Temporary tables, which are constructed by the SQL language processor when it performs its job of executing queries and data modifications. These relations are then thrown away and not stored.
