# Simple Sqlite3

A simple library for interacting with sqlite3 in Python. It is designed for use in web apps, which means that it commits and closes the database connection on each query.

## What/Why?

I found that I was reusing this code enough when working with flask and bottle that it made sense to turn it into a standalone package. This is not meant to be a comprehensive library for interacting with sqlite3, it's simply meant to streamline the process.

## How?

All interaction occurs through a `Database` class:

```py
from simple_sqlite3 import Database
```

The database object can be initialised by passing a path to the sqlite3 file, and an optional schema to initialise with:

```py
db = Database('my_database.db', schema_file='schema.sql')
```

**Note**: You can opt to initialise the database later using the `init_db` method, see the API below.

You can choose to run a single query, using the `query` method:

```py
rows = db.query('SELECT * FROM table_of_stuff')

word = 'aardvark'
row_id = db.query('INSERT INTO dictionary (word) VALUES (?)', (word,))
```

This will return a list of rows that have been retrieved, or a row id for queries that do not retrieve rows.

It is also possible to run multiple queries through the `many` method:

```py
words = (
    ('apple',),
    ('pear',),
    ('melon',)
)

db.many('INSERT INTO dictionary (word) VALUES (?)', words)
```

This will not return anything.

For more information on how SQL queries work in Python please see the [docs](https://docs.python.org/3.5/library/sqlite3.html).

## API

### Database(*db_file*, *schema_file=None*)

The main Database object to handle interactions with the sqlite3 database. Takes the path to the database file, and optionally a schema to initialise with.

- `db_file`: A path to an sqlite3 database file, relative to the executing directory or absolute.
- `schema_file` (*optional*): A schema file that will be used to initialise the database. This must be an SQL file (i.e. a file containing a series of SQL statements).

Returns a database (`db`) object with which all future interactions occur.

### Database.init_db(*schema_file*)

Used to initialise the database after it has been created, using a given schema file.

- `schema_file`: An SQL schema file, identical to the one that may be passed to the `Database` constructor above.

### Database.query(*querystring*, *args=()*)

Performs queries on the database, based on an SQL querystring and given arguments, and returns results.

- `querystring`: An SQL query in string form.
- `args` (*optional*): Arguments to be interpolated into the querystring, must be a tuple.

Return objects may either be a list of retrieved rows, for `SELECT` queries, or the id of a created/modified row for write-based queries (`INSERT`, `UPDATE`, etc.). Retrieved rows will be in the form of an [sqlite3.Row](https://docs.python.org/3.5/library/sqlite3.html#row-objects) object, which allows access to values via index or column key:

```py
result = db.query('SELECT id, word FROM dictionary')[0]
first_row = result[0]

id = first_row[0]
word = first_row['word']
```

### Database.many(*querystring*, *args=()*)

Works in the same way as the `query` method, but performs the given querystring multiple times for different sets of arguments.

- `querystring`: An SQL query in string form.
- `args` (*optional*): An iterable (e.g. list, tuple) of sets of arguments, where each set must be given as a tuple.

This method will not return anything. Here is an example of what is meant by an iterable of arguments:

```py
words = [
    ('apple', 'fruit'),
    ('pear', 'fruit'),
    ('cucumber', 'vegetable')
]

db.many('INSERT INTO five_a_day (food, type) VALUES (?, ?)', words)
```
