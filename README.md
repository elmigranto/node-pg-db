# Install

    $ npm install pg-db --save

# Usage

    // Create using default connection config of process.env.DATABASE_URL:
    var db = require('pg-db')();

    db.query('SELECT foo, bar, baz FROM some_table', function(err, rows){
      if( err ) return console.error('Err: %s', err);
      console.log('Rows: %j', rows);
    });

# Features
* Convenient wrapper functions - *callbacks for single row, multiple rows, or update row counts*
* Automatically return connections to the pool - *no need to call `client.done()`*
* Named parameters - *`... WHERE foo = :foo` instead of `... WHERE foo = $1`*
* Transactions - *automatic and transparent!*

# Transactions
Transactions are implemented using [domains](http://nodejs.org/api/domain.html). This allows the same node-postgres `client` object to be used by separate parts of your application without having to manually pass it as an argument.

    db.tx.series([
      async.apply(Foo.update, foo),
      async.apply(Audit.create, foo)
    ], cb);

# API

## Query API
If a transaction is in progress then all functions of the Query API will automatically use the connection `client` for the transactions.

If no transaction is in progress then a random connection will be retrieved from the pool of connections. After execution completes the connection will be returned to the pool.

If an error occurrs then by default the pool wil be instructed to destroy the connection. Internally, this is done by invoking `done(err)`.


### query(sql, [params], function cb(err, rows))
Execute SQL with the optional parameters and invoke the callback with the result rows.

This function is intended to be used with SQL that returns back a set of rows such as a `SELECT ...` statement. If no rows are returned then the callback will be invoked with an empty array.

### queryOne(sql, [params], function cb(err, row))
Execute SQL with the optional parameters and invoke the callback with the first result row.

This function is intended to be used with SQL that returns back a single row. If no rows are returned then the callback is invoked with a `null` value for row. If more than one row is returned then the callback is invoked with an `Error`.

__NOTE:__ *This function will return an `Error` if multiple rows are returned. This is intentional as it probably means your SQL is wrong.*

### update(sql, [params], function cb(err, rowCount))
Execute SQL with the optional parameters and invoke the callback with the number of rows that were modified.

This function is intended to be used with SQL that performs DML (e.g. `INSERT`, `UPDATE`, `DELETE`).

### execute(sql, [params], function cb(err, result))
Execute SQL with the optional parameters and invoke the callback with the raw result object returned by node-postgres.

This function is used internally by `query`, `queryOne`, and `update`. It's useful when you'd like to use both the `rowCount` and `rows`. Otherwise it's probably more convenient to use one of the other functions.

## Transaction API

### tx(function task(cb), function cb(err, result))
Executes a task in a transaction and invokes the callback with the result of the task.

### tx.series(tasks, function cb(err, results))
Convenience wrapper for executing a series of tasks within a transaction.

Internally this executes the tasks by calling [async.series](https://github.com/caolan/async#seriestasks-callback).

### tx.parallel(tasks, function cb(err, results))
Convenience wrapper for executing tasks in parallel within a transaction.

Internally this executes the tasks by calling [async.parallel](https://github.com/caolan/async#parallel).

### tx.auto(tasks, function cb(err, results))
Convenience wrapper for executing multiple tasks that depended on each other within a transaction.

Internally this executes the tasks by calling [async.auto](https://github.com/caolan/async#auto).

### tx.waterfall(tasks, function cb(err, results))
Convenience wrapper for executing tasks in a waterfall, passing each result to the next task, within a transaction.

Internally this executes the tasks by calling [async.waterfall](https://github.com/caolan/async#waterfall).

### tx.query(sql, [params], function cb(err, rows))
Ensure we're running within a transaction and execute the command.

If no transaction is in progress then the callback is invoked with an error. Otherwise this behaves exactly like the non-tx version.

### tx.queryOne(sql, [params], function cb(err, row))
Ensure we're running within a transaction and execute the command.

If no transaction is in progress then the callback is invoked with an error. Otherwise this behaves exactly like the non-tx version.

### tx.update(sql, [params], function cb(err, rowCount))
Ensure we're running within a transaction and execute the command.

If no transaction is in progress then the callback is invoked with an error. Otherwise this behaves exactly like the non-tx version.

### tx.execute(sql, [params], function cb(err, result))
Ensure we're running within a transaction and execute the command.

If no transaction is in progress then the callback is invoked with an error. Otherwise this behaves exactly like the non-tx version.
