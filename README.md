# README

## About

SQLite Simply provides a set of simple tools for interacting with a SQLite database using a database-first model.

It consists of the following classes:

* **SQLiteDatabase**: methods for connecting, querying, and executing SQL statements against a SQLite database
* **SQLiteHelper**: methods for reading and converting C# datatypes to/from SQLite affinity types

## Getting Started

General usage flow:

1. Instantiate the `SQLDatabase` class with a connection to a database file. Example:

	``` csharp
	var database = new SQLiteDatabase("Data Source=/data/mydatabase.db");
	```

  
2. Create a SQL statement with the `CreateStatement()` method. Example:

	``` csharp
	database.CreateStatement("SELECT FirstName FROM Users WHERE ID = @id");
	```

   
3. If needed, add parameters using the `AddParameter()` method.

	``` csharp
	database.AddParameter("@id", userID);
	```


4. Execute the desired type of operation using the provided methods (query, insert, execute). Wrap this in a `using` block to create the connection:

	``` csharp
	using (var connection = database.CreateConnection())
	{
		SqliteDataReader reader = database.Query(connection);
		// do something with the reader
	}
	```

## `SQLiteDatabase` Methods

* **CreateStatement(string)**: sets up a new SQL statement and clears parameters; this SQL will be used by the query, insert, and execute methods
* **AppendStatement(string)**: adds the given string to the SQL statement created with `CreateStatement()`
* **AddParameter(string, object)**: attaches a parameterized value to the SQL command
* **Query(SqliteConnection)**: executes a query against the database; returns a SqliteDataReader
* **Insert(SqliteConnection)**: executes an insert statement against the database; returns an integer representing the _rowid_ of the new row
* **Execute(SqliteConnection)**: executes arbitrary SQL against the database; returns void

## Database IDs &amp; Type Warnings

_SQLiteDatabase_ assumes that your primary keys is the numeric type (i.e. integers). The `Insert()` method will always attempt to return an integer _rowid_.

When inserting data, the SQLite function `last_insert_rowid()` is used to get the latest _rowid_ value, which has some [known potential downsides with concurrency (Stack Overflow)](https://stackoverflow.com/questions/2127138/how-to-retrieve-the-last-autoincremented-id-from-a-sqlite-table). Future versions of this library will attempt to address this.

Feel free to fork this repo and modify the insert logic as you see fit.

## Example Operations

The following examples assume that you instantiate the `SQLDatabase` class with a connection to a SQLite database:

``` csharp
var database = new SQLiteDatabase("/data/mydatabase.db");
```

### SELECT

``` csharp
database.CreateStatement(@"
	SELECT
		ID,
		FirstName,
		Birthdate,
	FROM
		Users
	WHERE
		Active = 1
		AND State = @state");

database.AddParameter("@state", "Ohio");

using (var connection = _sqlite.CreateConnection())
{
    var reader = _sqlite.Query(connection);

    while (reader.Read())
    {
        User user = new User();

        user.ID = SQLiteHelper.GetID(reader["UserID"]);
        user.Firstname = SQLiteHelper.GetString(reader["CategoryID"]);
        user.Birthdate = SQLiteHelper.GetDateTime(reader["Birthdate"]);
    }
}
```

### INSERT

``` csharp
database.CreateStatement(@"
	INSERT INTO Users (FirstName, Email, Birthday)
	VALUES (@firstname, @email, @birthdate)");

database.AddParameter("@firstname", firstName);
database.AddParameter("@email", emailAddress);
database.AddParameter("@birthdate", SQLiteHelper.FormatDateTimeForSQL(birthdate));

using (var connection = _sqlite.CreateConnection())
{
    int newID = _sqlite.Insert(connection);
}
```

### UPDATE

``` csharp
database.CreateStatement(@"
	UPDATE Users
	SET
		FirstName = @firstname,
		Email = @email,
		Birthdate = @birthdate
	WHERE
		ID = @userid");

database.AddParameter("@userid", userID)
database.AddParameter("@firstname", firstName);
database.AddParameter("@email", emailAddress);
database.AddParameter("@birthdate", SQLiteHelper.FormatDateTimeForSQL(birthdate));

using (var connection = _sqlite.CreateConnection())
{
    _sqlite.Execute(connection);
}
```

### DELETE

``` csharp
database.CreateStatement(@"
	DELETE FROM Users
	WHERE ID = @userid");

database.AddParameter("@userid", userID)

using (var connection = _sqlite.CreateConnection())
{
    _sqlite.Execute(connection);
}
```

## Datatypes

## Dependencies

Microsoft.Data.Sqlite (8.0.6)

## License
