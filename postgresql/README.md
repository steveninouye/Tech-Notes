# PostgreSQL

## AS
Allows to assign alias for a column
```
# gets the name column and renames the column as employee_name
SELECT name AS employee_name FROM company;
```

## COUNT
```
SELECT count(*) FROM company;
```

## MAX / MIN
```
SELECT MAX(age) FROM company;

SELECT MIN(age) FROM company;
```

## SUM
```
SELECT SUM(salary) FROM company;
```

## AVG
Average
```
SELECT AVG(salary) FROM company;
```

## CURRENT_TIMESTAMP
```
SELECT CURRENT_TIMESTAMP;
```

## WHERE
Filters results of `SELECT` method with comparison operators
```
SELECT name FROM company
WHERE age = 34;

SELECT name FROM company
WHERE age!=34;

SELECT * FROM company
WHERE age > 34;

SELECT * FROM company
WHERE age < 34;

SELECT * from company
WHERE age >= 34;

SELECT * FROM company
WHERE age <= 34;
```

## AND
```
SELECT * FROM company
WHERE age > 33 AND salary > 2000;
```

## OR
```
SELECT * FROM company
WHERE age > 33 OR salary>2000;
```

## IS NULL
```
SELECT * FROM company
WHERE salary IS NULL;
```

## LIKE
```
# name column values begin with the letter P
SELECT * FROM company
WHERE name LIKE 'P%';

# name column values end with the letter a
SELECT * FROM company
WHERE name LIKE '%a';

# the letter a is anywhere in the name column
SELECT * FROM company
WHERE name LIKE '%a%';
```

## IN
```
# gets results that age only equal to 20, 34, and 40
SELECT * FROM company
WHERE age IN('20','34','40');
```

## BETWEEN
```
SELECT * FROM company
WHERE age BETWEEN 23 AND 34;
```

## LIMIT
Limits the amount of results returned
```
SELECT * FROM company
LIMIT 4 OFFSET 4
```

## OFFSET
Will skip the number of offset and return the remaining results
```
SELECT * FROM company
LIMIT 4 OFFSET 4
```

## GROUP BY
Groups result by a certain field/column
* Must have aggregate operator
```
# return results from the company table
# group them by age
SELECT age, count(*) FROM company
GROUP BY age
```

## HAVING
Will return results that meet `HAVING` requirements
* Must have a `GROUP BY` clause
```
SELECT address, count(*) FROM company
GROUP BY address
HAVING MAX(salary)>2000
```
