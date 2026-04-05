# SimpleMySQL
An ultra simple wrapper for `mysql-connector-python` with very basic functionality.

##### Authors
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
    <tr>
        <td align="center">
            <a href="https://github.com/lilkingjr1">
                <img src="https://avatars.githubusercontent.com/u/4533989" width="50px;" alt=""/><br /><sub><b>David Wolfe</b></sub>
            </a>
            <br />
            <a href="https://github.com/lilkingjr1/simplemysql/commits?author=lilkingjr1" title="Codes">💻</a>
            <a href="https://github.com/lilkingjr1/simplemysql/commits?author=lilkingjr1" title="Maintains">🔨</a>
        </td>
        <td align="center">
            <a href="https://github.com/milosb793">
                <img src="https://avatars.githubusercontent.com/u/5012355" width="50px;" alt=""/><br /><sub><b>Milosh Bolic</b></sub>
            </a>
            <br />
            <a href="https://github.com/knadh/simplemysql/commits?author=milosb793" title="Codes">💻</a>
            <a href="https://github.com/knadh/simplemysql/commits?author=milosb793" title="Contributor">💡</a>
        </td>
        <td align="center">
            <a href="https://github.com/knadh">
                <img src="https://avatars.githubusercontent.com/u/547147" width="50px;" alt=""/><br /><sub><b>Kailash Nadh</b></sub>
            </a>
            <br />
            <a href="https://github.com/knadh/simplemysql/commits?author=knadh" title="Codes">💻</a>
            <a href="https://github.com/knadh/simplemysql/commits?author=knadh" title="Original Creator">⭐</a>
            <a href="https://github.com/knadh/simplemysql/commits?author=knadh" title="Retired from Development">💤</a>
        </td>
    </tr>
</table>
<!-- markdownlint-enable -->
<!-- prettier-ignore-end -->
Licensed under GNU GPL v2

## Installation
With pip3

```pip3 install git+https://github.com/lilkingjr1/simplemysql```

Or from the source

```python setup.py install```

# Usage
## For normal connection:
```python
from simplemysql import SimpleMysql

db = SimpleMysql(
	host="127.0.0.1",
	port="3306",
	db="mydatabase",
	user="username",
	passwd="password",
	autocommit=False, # optional kwarg
	keep_alive=True # try and reconnect timedout mysql connections?
)
```
## For SSL Connection:
```python
from simplemysql import SimpleMysql

db = SimpleMysql(
	host="127.0.0.1",
	port="3306",
	db="mydatabase",
	user="username",
	passwd="password",
	ssl = {'cert': 'client-cert.pem', 'key': 'client-key.pem'},
	keep_alive=True # try and reconnect timedout mysql connections?
)
```
## Usage example:
```python
# create table, if it doesn't already exist
db.query("CREATE TABLE IF NOT EXISTS books (id INT AUTO_INCREMENT PRIMARY KEY, type VARCHAR(255), name VARCHAR(255), price DECIMAL(10, 2), year INT)")

# insert a record to the "books" table
db.insert("books", {"type": "paperback", "name": "Time Machine", "price": 5.55, "year": 1997})

# retrieve a single record that contains the "name" attribute with a "year" of 1997 from the "books" table
book = db.getOne("books", ["name"], ("year=%s", [1997]))

# access the "name" attribute from the returned record
print("The book's name is " + book['name'])
```

# Query methods
| Returns | Methods |
| ----------- | ------- |
| int | [insert(table, record{})](#inserttable-record) |
| int | [insertBatch(table, rows{})](#insertbatchtable-rows) |
| int | [update(table, row{}, condition[])](#updatetable-row-condition) |
| int | [insertOrUpdate(table, row{}, key)](#insertorupdatetable-row-key) |
| dict | [getOne(table, fields[], where[], order[], limit[])](#getonetable-fields-where-order-limit) |
| dict[] | [getAll(table, fields[], where[], order[], limit[]))](#getalltable-fields-where-order-limit) |
| dict[] | [leftJoin(tables(2), fields(), join_fields(2), where=[], order=[], limit=[])](#leftJointables2-fields-join_fields2-where-order-limit) |
| int | [delete(table, fields[], condition[], order[], limit[])](#deletetable-fields-condition-order-limit) |
| tuple[] | [call(procedure, params)](#callprocedure-params) |
| int | [lastId()](#lastid) |
| str | [lastQuery()](#lastquery) |
| MySQLdb.Cursor | [query(string)](#querystring) |
| void | [commit()](#commit) |

## insert(table, record{})
Inserts a single record into a table. Returns number of rows affected.

```python
db.insert("food", {"type": "fruit", "name": "Apple", "color": "red"})
db.insert("books", {"type": "paperback", "name": "Time Machine", "price": 5.55})
```

## insertBatch(table, rows{})
Insert Multiple values into table. Returns number of rows affected.

```python
# insert multiple values in table
db.insertBatch("books", [{"discount": 0},{"discount":1},{"discount":3}])
```

## update(table, row{}, condition[])
Update one more or rows based on a condition (or no condition). Returns number of rows affected.

```python
# update all rows
db.update("books", {"discount": 0})

# update rows based on a simple hardcoded condition
db.update(
	"books",
	{"discount": 10},
	["id=1"]
)

# update rows based on a parametrized condition
db.update(
	"books",
	{"discount": 10},
	[f"id={id} AND year={year}"]
)
```

## insertOrUpdate(table, row{}, key)
Insert a new row, or update if there is a primary key conflict. Returns number of rows affected.

```python
# insert a book with id 123. if it already exists, update values
db.insertOrUpdate(
	"books",
	{"id": 123, type": "paperback", "name": "Time Machine", "price": 5.55},
	"id"
)
```

## getOne(table, fields[], where[], order[], limit[])
Get a single record from a table given a condition (or no condition). The resultant row is returned as a single dictionary. `None` is returned if no record can be found.

```python
book = db.getOne("books", ["id", "name"])

print(f"Book {book['name']} has ID of {book['id']}")
```

```python
# get a row based on a simple hardcoded condition
book = db.getOne("books", ["name", "year"], ("id=1"))
```

```python
# get all columns of first result/row of a table
book = db.getOne("books", ["*"])
```

## getAll(table, fields[], where[], order[], limit[])
Get multiple records from a table given a condition (or no condition). The resultant rows are returned as a list of dictionaries. `None` is returned if no record(s) can be found.

```python
# get multiple rows based on a parametrized condition
books = db.getAll(
	"books",
	["id", "name"],
	("year > %s and price < %s", [year, 12.99])
)

print("All books that match conditions:\n")
for record in books:
	print(f"{record['name']}\n")
```

```python
# get multiple rows based on a parametrized condition with an order and limit specified
books = db.getAll(
	"books",
	["id", "name", "year"],
	("year > %s and price < %s", [year, 12.99]),
	["year", "DESC"],	# ORDER BY year DESC for descending (ASC for ascending)
	[0, 10]			# LIMIT 0, 10
)
```

***Note: `getOne()` and `getAll()` work with View Objects as well:***

```python
# get all rows and columns of a view
discounted_books = db.getAll(
	"discounted_books_view",
	["*"]
)
```

## leftJoin(tables(2), fields()[], join_fields(2), where=[], order=[], limit=[])
Get multiple records, for multiple fields, spanning two tables, given a condition (or no condition) against both tables and two joining fields that would share the same value between tables. The resultant rows are returned as a list of dictionaries. `None` is returned if no record(s) can be found.

```python
# get multiple records for multiple fields spanning two tables that share a common field, based on a parametrized condition
book_sales = db.leftJoin(
    ("books", "transactions"),
    (
        ["edition"], # "books" fields
        ["fname", "date"] # "transactions" fields
    ),
    ("id", "book_id"), # same id number in both tables
    ("books.name=%s and transactions.fname=%s", [book_name, first_name])
)

print(f"All sales that match book {book_name} and customer {first_name}:\n")
for sale in book_sales:
	print(f"{sale['fname']} purchased {sale['edition']} edition on {sale['date']}")
```

## delete(table, fields[], condition[], order[], limit[])
Delete one or more records based on a condition (or no condition). Returns number of rows affected.

```python
# delete all rows
db.delete("books")

# delete rows based on a condition
db.delete("books", ("price > %s AND year < %s", [25, 1999]))
```

## call(procedure, params[])
Calls/runs a stored procedure by name in the connected database. All return values of the procedure are fetched and returned as a tuple list. `None` is returned if the procedure does not return anything.

```python
# call a stored procedure in the database
results = db.call("DeleteOldBooksListNew", [1970, 2000])

# print procedure return values
for book in results:
    print(f'Title: "{book(0)}", Year: {book(1)}')

## lastId()
Get the last insert ID.

```python
# get the last insert ID
db.lastId()
```

## lastQuery()
Get the last query executed.

```python
# get the SQL of the last executed query
db.lastQuery()
```

## query(string)
Run a raw SQL query. The MySQLdb cursor is returned.

```python
# run a raw SQL query
db.query("DELETE FROM books WHERE year > 2005")
```
```

## commit()
Insert, update, and delete operations on transactional databases such as innoDB need to be committed.

```python
# Commit all pending transaction queries
db.commit()
```
