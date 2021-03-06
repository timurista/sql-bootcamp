# Sql Bootcamp

Notes on SQL course

## From Spreadhseet to Databases

use cases Spreadsheets
- quickly need to chart something else
- reasonable sizes
- untrained people to work 

Databases
- data integrity (change data)
- quickly combine data sets

## SELECT FROM
captialize select by convention for easier reading?
in real case usually you use 100 not * to get all data

## SELECT with DISTINCT
in table, column may contain duplicate values
you jsut want distinct values

SELECT DISTINCT column1, column2
FROM tablename

## SELECT with WHERE
SELECT column1, col2...
FROM table_name
WHERE conditions;

```
SELECT customer
WHERE first_name = 'Tim' AND last_name='John'
basic logical operators, = > < AND != OR
```

## COUNT
COUNT statement, counts where condition was true
```
SELECT COUNT(DISTINCT amount) FROM payment;
```

## LIMIT
limit rows back
```
SELECT * FROM customer LIMIT 1;
```

## ORDER BY
sort rows in either ascending or descending
```
SELECT first_name, last_name FROM customer ORDER BY first_name ASC;

// also by multiple columns
SELECT first_name, last_name FROM customer ORDER BY first_name ASC, last_name DESC;
```

## BETWEEN
value BETWEEN low AND high
```
SELECT customer_id, amount, payment_date FROM payment
WHERE payment_date BETWEEN '2007-02-07' AND '2007-02-15';

// you can use numbers or strings
```

## IN
to check if value matches any value of list. List of values can be set of select statements

value IN (SELECT value FROM table)

IN is shortcut for nested ORs

```
SELECT customer_id, rental_id, return_date 
FROM rental 
WHERE customer_id NOT IN(1,2,13) 
ORDER BY return_date DESC;
```

## LIKE
how to find a name say 
```
SELECT first_name, last_name
FROM customer
WHERE first_name LIKE 'Jen%'
```
uses pattern matching in the second
string with wild card characters
Perent % for any sequence of characters, underscore _ for matching any single character. So _ is 1 char, __ is 2 chars. 

Post gres allows you to do ILIKE, so you can ignore string
ILIKE ignores case

### CHALLENGES
1. How many payment transactions were greater than $5.00?
ans: SELECT COUNT(*) FROM payment WHERE amount > 5;
3,618

2. how many actors have a first name that starts with P?
ans: SELECT COUNT(*) FROM actor WHERE first_name LIKE 'P%';
return 5

3. How many unique districts are our cuspstomers from?
ans: SELECT COUNT( DISTINCT district) FROM address;

4. List of names from those distinct districts in last question
SELECT DISTINCT district FROM address;

5. How many films have a rating of R and cose replacement between 5 and 15?
```
SELECT COUNT(*) FROM film 
WHERE rating = 'R' 
AND replacement_cost BETWEEN 5 AND 15;
```

6. How many films have the word Truman somewhere in the title?
```
SELECT COUNT(title) FROM film 
WHERE title LIKE '%Truman%';
```

## MIN MAX AVG SUM
aggregate functions
```
SELECT ROUND( AVG(amount), 2) FROM payment; // 4.20
SELECT COUNT(*) FROM payment WHERE amount=0.00; // MIN

```

## GROUP BY
divides rows returned into groups
aggregates all info into a single value
```
SELECT col1, aggregate function
FROM table
GROUP BY col1

SELECT customer_id, SUM(amount)
FROM payment
GROUP BY customer_id;

// select column you are grouping by

SELECT staff_id, COUNT(payment)
FROM payment
GROUP BY staff_id;

// counts rows that statement is returning back
// takes all instances of staff id and increments by count

SELECT rating, AVG(rental_rate) 
FROM film
GROUP BY rating;
```

### challenges
We have two staff members id 1 and 2
give bonus to member with most payments

how many payments each staff member handle?
how much was total amount?

```
SELECT staff_id, COUNT(staff_id), SUM(staff_id)
FROM payment
GROUP BY staff_id ORDER BY COUNT(staff_id) DESC;
```

2. Corporate HQ want to know avg replacemnt cost of movies by rating.

```
SELECT rating, ROUND(AVG(replacement_cost), 2)
FROM film
GROUP BY rating ORDER BY AVG(replacement_cost) DESC;
```

3. send coupons to 5 customers who have spend the most amount of money.
get customer ids of top 5 spenders

```
SELECT customer_id, SUM(amount)
FROM payment 
GROUP BY customer_id ORDER BY SUM(amount) DESC
LIMIT 5;
```

## Having
like where clause but for group by condition
examples

```
SELECT rating, AVG(rental_rate)
FROM film
WHERE rating in ('R', 'G', 'PG')
GROUP BY rating
HAVING AVG(rental_rate) < 3; 
```
select and use where clause to get rating no aggregate function
then having to get having avg

challenge
1. customer has at least a total of 40 transaction payments
what customers by customer_id are eligible?

```
SELECT customer_id, COUNT(amount)
FROM payment
GROUP BY customer_id
HAVING COUNT(amount) >= 40;
```

2. when grouped by rating, what movie ratings have an avg rental duration of more than 5 days
```
SELECT rating, ROUND(AVG(rental_duration), 2)
FROM film
GROUP BY rating
HAVING AVG(rental_duration) > 5;
```

[See assessment 1](assessments/assessment1.md)


# Part 2

## AS
for joins, asigns alias name to columns

## JOINS
relate data in 1 table to data in another
INNER JOIN, OUTER JOIN, SELF JOIN

tables A and B
A 
is primary k is integer
c1 is fullname

B
key is pkb
c2 is custmer name
foreign key is a

```
SELECT A.pka, A.c1, B.pkb, B.c2
FROM A
INNER JOIN B ON A.pka = B.fka
```

INNER JOIN is what is in A and B, intersection of A and B
```
SELECT 
customer.customer_id, 
first_name, 
last_name,
amount,
payment_date
FROM customer
INNER JOIN payment ON customer.customer_id = payment.customer_id;
```

Only need first_name instead of customer, because it is only in a single table
```
SELECT 
customer.customer_id, 
first_name, 
last_name,
amount,
payment_date
FROM customer
INNER JOIN payment ON customer.customer_id = payment.customer_id
WHERE customer.customer_id = 2 //
ORDER BY first_name; // use order by
```

INNER JOIN, you can just write JOIN <-- it is the default

```
SELECT store_id, title, COUNT(title)
FROM inventory
JOIN film ON inventory.film_id = film.film_id
GROUP BY title, store_id ORDER BY store_id, COUNT(title) DESC;
```

```
SELECT title, COUNT(title) AS copies_at_store1
FROM inventory
JOIN film ON inventory.film_id = film.film_id
WHERE store_id = 1
GROUP BY title;
```

gets language name and title of film
```
SELECT film.title, language.name AS movie_language
FROM film
JOIN language ON language.language_id = film.language_id;
```

In real workplace, people will remove AS, and leave space because it works.


## Inner Join
records only in Table A and Table B
Intersection of Table A and Table B (also default JOIN)

## FULL OUTER JOIN
Table A + Table B
missing records produce a null
takes right and left table and puts in nulls

## LEFT OUTER JOIN
complete set of records from A
first table A returns all rows from table A
if no match, return with null

## RIGHT OUTER JOIN
table B is included by not from table A

## LEFT OUTER JOIN with WHERE
check if records are null
```
SELECT * FROM TableA
LEFT OUTER JOIN TableB
ON TableA.name = TableB.name
WHERE TableB.id IS null
```
so you then get those things in A but Not in B

## FULL OUTER JOIN with WHERE
only records where they don't match up
```
SELECT * FROM TableA
FULL OUTER JOIN TableB
ON TableA.name = TableB.name
WHERE TableA.id IS null
OR TableB.id IS null
```

Find all films which are not in inventory
```
SELECT film.film_id, film.title, inventory_id
FROM film
LEFT OUTER JOIN inventory ON inventory.film_id = film.film_id
WHERE inventory.film_id IS NULL;
```


## UNION
both same number of cols in tables must be returned
and select col 1 must be same col data types
removes duplicates

```
SELECT * FROM salesa
UNION
SELECT * FROM salesb
```
UNION ALL --> to get all results even the duplicated rows


## timestamps
- extract( unit from date)
you can extract day, dow, epoch, hour etc.

```
SELECT SUM(amount) As total, extract(month from payment_date) AS month
FROM payment
GROUP BY month
ORDER BY total DESC;
```
## string and math operators
postgressql string functions and operators
```
SELECT first_name || ' ' || last_name AS full_name
FROM customer;
```

## Subquery
allows us to use query in a query

```
SELECT title, rental_rate
FROM film
WHERE rental_rate > (SELECT AVG(rental_rate) FROM film);
```

```
SELECT film_id, title
FROM film
WHERE film_id IN

(SELECT inventory.film_id 
FROM rental
JOIN inventory ON inventory.inventory_id = rental.inventory_id
WHERE return_date BETWEEN '2005-05-29' AND '2005-05-30');
```

above, use single string return comparison operator

## self join
col `employee_name` and `employee_location`
this is more efficient to join table to itself than to use subquery.

```
Select e1.employee_name
FROM employee AS e1, employee AS e2 // self join
WHERE
e1.employee_location = e2.employee_location
AND e2.employee_name="Joe1";
```
self joins don't use JOIN terminology

also can explicitly use JOIN
```
FROM customer AS a
JOIN customer AS b
ON a.first_name = b.last_name;
```

## Assessment 2
[See assessment 2](assessments/assessment2.md)


# Drop Create Tables, etc

## DataTypes PostgresSql
- Boolean
- Character
- Number
Temporal
- Special like: Array

## Boolean
converting to true --> yes, 1, etc.
converting to false --> no, 0, etc

## Character
fixed-length character strings: char(n)
if string is shorter than length of column,
  then pad with spaces
if string is longer, error is thrown

### why use this?
  - enforce it so user cannot mess up ID code

- variable-length char strings: varchar(n). Postgres does not padd with spaces if it is shorter than length n.

## Number
Integers, Floating Point Number
smallint -/+ 3276832767 
Integer 4-btye
Serial <-- postgres populates the db with it

Float. Preceision to n, up to max of 8 bytes
real or float8 is double-precision (8-byte) floating point

numeric(p,s) real number with s nmber after the decimal point.
numeric(p,) is the exact number

Temporal,
 date
time
interval
timestamp
timestamptz 


## PRIMARY AND FOREIGN KEYS
primary key constraints
only one primary key 
constraints = PRIMAY KEY

foreign key is a field or cols which is primary key of another table
REFERENCING TABLE

you can inherit from existing table

`CREATE TABLE table_name (column name TYPE column_constraint table_contstraint);`

## CONSTRAINTS
UNIQUE across whole table
null value is unique
NOT NULL
PRIMARY KEY == NOT NULL AND UNIQUE

- CHECK enables condition when you update data
- REFERENCES foreign key constraint

