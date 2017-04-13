# class *Database*

- [new Database()](#new-databasepath-options)
- [Database#prepare()](#preparestring---statement)
- [Database#transaction()](#transactionarrayofstrings---transaction)
- [Database#pragma()](#pragmastring-simplify---results)
- [Database#checkpoint()](#checkpointforce---number)
- [Database#close()](#close---this)
- [Database#open](#get-open---boolean)
- [Database#name](#get-name---string)
- [Database#memory](#get-memory---boolean)
- [Database#readonly](#get-readonly---boolean)

### new Database(*path*, [*options*])

Creates a new database connection. If the database file does not exist, it is created. This happens synchronously, which means you can start executing queries right away.

If `options.memory` is `true`, an in-memory database will be created, rather than a disk-bound one. Default is `false`.

If `options.readonly` is `true`, the database connection will be opened in readonly mode. Default is `false`.

### .prepare(*string*) -> *Statement*

Creates a new prepared [`Statement`](#class-statement) from the given SQL string.

### .transaction(*arrayOfStrings*) -> *Transaction*

Creates a new prepared [`Transaction`](#class-transaction) from the given array of SQL strings.

*NOTE:* [`Transaction`](#class-transaction) objects cannot contain read-only statements. In `better-sqlite3`, transactions serve the sole purpose of batch-write operations. For read-only operations, use regular [prepared statements](#preparestring---statement). This restriction may change in the future.

### .pragma(*string*, [*simplify*]) -> *results*

Executes the given PRAGMA and return its result. By default, the return value will be an array of result rows. Each row is represented by an object whose keys correspond to column names.

Since most PRAGMA statements return a single value, the `simplify` option is provided to make things easier. With this option, only the first column of the first row will be returned.

```js
db.pragma('cache_size = 32000');
var cacheSize = db.pragma('cache_size', true); // returns the string "32000"
```

The data returned by `.pragma()` is always in string format. If execution of the PRAGMA fails, an `Error` is thrown.

It's better to use this method instead of normal [prepared statements](#preparestring---statement) when executing PRAGMA, because this method normalizes some odd behavior that may otherwise be experienced. The documentation on SQLite3 PRAGMA can be found [here](https://www.sqlite.org/pragma.html).

### .checkpoint([*force*]) -> *number*

Runs a [WAL mode checkpoint](https://www.sqlite.org/wal.html).

By default, this method will execute a checkpoint in "PASSIVE" mode, which means it might not perform a *complete* checkpoint if other processes are using the database at the same time. If the first argument is `true`, it will execute the checkpoint in "RESTART" mode, which ensures a complete checkpoint operation.

You only need to force checkpoints ("RESTART" mode) if you are accessing the database from multiple processes at the same time.

When the operation is complete, it returns a number between `0` and `1`, indicating the fraction of the WAL file that was checkpointed. For forceful checkpoints, this number will always be `1` unless there was no WAL file to begin with.

If execution of the checkpoint fails, an `Error` is thrown.

### .close() -> *this*

Closes the database connection. After invoking this method, no statements/transactions can be created or executed.

### *get* .open -> *boolean*

Returns whether the database is currently open.

### *get* .name -> *string*

Returns the string that was used to open the databse connection.

### *get* .memory -> *boolean*

Returns whether the database is an in-memory database.

### *get* .readonly -> *boolean*

Returns whether the database connection was created in readonly mode.

# class *Statement*

An object representing a single SQL statement.

- [Statement#run()](#runbindparameters---object)
- [Statement#get()](#getbindparameters---row)
- [Statement#all()](#allbindparameters---array-of-rows)
- [Statement#each()](#eachbindparameters-callback---undefined)
- [Statement#pluck()](#plucktogglestate---this)
- [Statement#bind()](#bindbindparameters---this)
- [Statement#source](#get-source---string)
- [Statement#returnsData](#get-returnsdata---boolean)

### .run([*...bindParameters*]) -> *object*

**(only on statements that do not return data)*

Executes the prepared statement. When execution completes it returns an `info` object describing any changes made. The `info` object has two properties:

- `info.changes`: The total number of rows that were inserted, updated, or deleted by this operation. Changes made by [foreign key actions](https://www.sqlite.org/foreignkeys.html#fk_actions) or [trigger programs](https://www.sqlite.org/lang_createtrigger.html) do not count.
- `info.lastInsertROWID`: The [rowid](https://www.sqlite.org/lang_createtable.html#rowid) of the last row inserted into the database. If the current statement did not insert any rows into the database, this number should be completely ignored.

If execution of the statement fails, an `Error` is thrown.

You can specify [bind parameters](#binding-parameters), which are only bound for the given execution.

### .get([*...bindParameters*]) -> *row*

**(only on statements that return data)*

Executes the prepared statement. When execution completes it returns an object that represents the first row retrieved by the query. The object's keys represent column names.

If the statement was successful but found no data, `undefined` is returned. If execution of the statement fails, an `Error` is thrown.

You can specify [bind parameters](#binding-parameters), which are only bound for the given execution.

### .all([*...bindParameters*]) -> *array of rows*

**(only on statements that return data)*

Similar to [`.get()`](#getbindparameters---row), but instead of only retrieving one row all matching rows will be retrieved. The return value is an array of row objects.

If no rows are found, the array will be empty. If execution of the statement fails, an `Error` is thrown.

You can specify [bind parameters](#binding-parameters), which are only bound for the given execution.

### .each([*...bindParameters*], callback) -> *undefined*

**(only on statements that return data)*

Similar to [`.all()`](#allbindparameters---array-of-rows), but instead of returning every row together, the `callback` is invoked for each row as they are retrieved, one by one.

After all rows have been consumed, `undefined` is returned. If execution of the statement fails, an `Error` is thrown and iteration stops.

You can specify [bind parameters](#binding-parameters), which are only bound for the given execution.

### .pluck([toggleState]) -> *this*

**(only on statements that return data)*

Causes the prepared statement to only return the value of the first column of any rows that it retrieves, rather than the entire row object.

You can toggle this on/off as you please:

```js
var stmt = db.prepare(SQL);
stmt.pluck(); // plucking ON
stmt.pluck(true); // plucking ON
stmt.pluck(false); // plucking OFF
```

### .bind([*...bindParameters*]) -> *this*

[Binds the given parameters](#binding-parameters) to the statement *permanently*. Unlike binding parameters upon execution, these parameters will stay bound to the prepared statement for its entire life.

This method can only be invoked before the statement is first executed. After a statement's parameters are bound this way, you may no longer provide it with execution-specific (temporary) bound parameters.

This method is primarily used as a performance optimization when you need to execute the same prepared statement many times with the same bound parameters.

### *get* .source -> *string*

Returns the source string that was used to create the prepared statement.

### *get* .returnsData -> *boolean*

Returns whether the prepared statement returns data.

# class *Transaction*

An object representing many SQL statements grouped into a single logical [transaction](https://www.sqlite.org/lang_transaction.html).

- [Transaction#run()](#runbindparameters---object-1)
- [Transaction#bind()](#bindbindparameters---this-1)
- [Transaction#source](#get-source---string-1)

### .run([*...bindParameters*]) -> *object*

Similar to [`Statement#run()`](#runbindparameters---object).

Each statement in the transaction is executed in order. Failed transactions are automatically rolled back.

When execution completes it returns an `info` object describing any changes made. The `info` object has two properties:

- `info.changes`: The total number of rows that were inserted, updated, or deleted by this transaction. Changes made by [foreign key actions](https://www.sqlite.org/foreignkeys.html#fk_actions) or [trigger programs](https://www.sqlite.org/lang_createtrigger.html) do not count.
- `info.lastInsertROWID`: The [rowid](https://www.sqlite.org/lang_createtable.html#rowid) of the last row inserted into the database. If the current transaction did not insert any rows into the database, this number should be completely ignored.

If execution of the transaction fails, an `Error` is thrown.

You can specify [bind parameters](#binding-parameters), which are only bound for the given execution.

### .bind([*...bindParameters*]) -> *this*

Same as [`Statement#bind()`](#bindbindparameters---this).

### *get* .source -> *string*

Returns a concatenation of every source string that was used to create the prepared transaction. The source strings are seperated by newline characters (`\n`).

# Binding Parameters

This section refers to anywhere in the documentation that specifies the optional argument [*`...bindParameters`*].

There are many ways to bind parameters to a prepared statement or transaction. The simplest way is with anonymous parameters:

```js
var stmt = db.prepare('INSERT INTO people VALUES (?, ?, ?)');

// The following are equivalent.
stmt.run('John', 'Smith', 45);
stmt.run(['John', 'Smith', 45]);
stmt.run(['John'], ['Smith', 45]);
```

You can also use named parameters. SQLite3 provides [4 different syntaxes for named parameters](https://www.sqlite.org/lang_expr.html), **three** of which are supported by `better-sqlite3` (`@foo`, `:foo`, and `$foo`). However, if you use named parameters, make sure to only use **one** syntax within a given [`Statement`](#class-statement) or [`Transaction`](#class-transaction) object. Mixing syntaxes within the same object is not supported.

```js
// The following are equivalent.
var stmt = db.prepare('INSERT INTO people VALUES (@firstName, @lastName, @age)');
var stmt = db.prepare('INSERT INTO people VALUES (:firstName, :lastName, :age)');
var stmt = db.prepare('INSERT INTO people VALUES ($firstName, $lastName, $age)');

stmt.run({
	firstName: 'John',
	lastName: 'Smith',
	age: 45
});
```

Below is an example of mixing anonymous parameters with named parameters.

```js
var stmt = db.prepare('INSERT INTO people VALUES (@name, @name, ?)');
stmt.run(45, {name: 'Henry'});
```