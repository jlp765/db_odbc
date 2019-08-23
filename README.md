# db_odbc
Nim ODBC library consistent with the nim db_xxxxxx libraries

Currently all queries are ANSI calls, not Unicode.

Tested odbc databases:
 * Oracle, 
 * Sybase, 
 * MSSqlvSvr, 
 * Firebird

## Examples

### Parameter substitution
 All ``db_*`` modules support the same form of parameter substitution.
 That is, using the ``?`` (question mark) to signify the place where a
 value should be placed. For example:

```Nim
  sql"INSERT INTO myTable (colA, colB, colC) VALUES (?, ?, ?)"
```

#### Opening a connection to a database
 
```Nim
  import db_odbc
  var db = open("localhost", "user", "password", "dbname")
  db.close()
```

#### Creating a table

```Nim
  db.exec(sql"DROP TABLE IF EXISTS myTable")
  db.exec(sql("""CREATE TABLE myTable (
                 id integer,
                 name varchar(50) not null)"""))
```

#### Inserting data

```Nim
  db.exec(sql"INSERT INTO myTable (id, name) VALUES (0, ?)",
             "Andreas")
```

#### Large example

```Nim

  import db_odbc, math

  var theDb = open("localhost", "nim", "nim", "test")

  theDb.exec(sql"Drop table if exists myTestTbl")
  theDb.exec(sql("create table myTestTbl (" &
      " Id    INT(11)     NOT NULL AUTO_INCREMENT PRIMARY KEY, " &
      " Name  VARCHAR(50) NOT NULL, " &
      " i     INT(11), " &
      " f     DECIMAL(18,10))"))

  theDb.exec(sql"START TRANSACTION")
  for i in 1..1000:
    theDb.exec(sql"INSERT INTO myTestTbl (name,i,f) VALUES (?,?,?)",
                  "Item#" & $i, 
                  i, 
                  sqrt(i.float))
  theDb.exec(sql"COMMIT")

  for x in theDb.fastRows(sql"select * from myTestTbl"):
    echo x

  let id = theDb.tryInsertId(sql"INSERT INTO myTestTbl (name,i,f) VALUES (?,?,?)",
                                "Item#1001", 
                                1001, 
                                sqrt(1001.0))
  echo "Inserted item: ", theDb.getValue(sql"SELECT name FROM myTestTbl WHERE id=?", id)

  theDb.close()
```

=====================================================================
## History
This initially started in the Nim standard Library, but was removed because it lacks stability (not extensively tested against many database types.
For initial history, search the Issues and PRs of Nim.

1a5bde28edeb2c3ffeeb73ef080f2e6e99f21761  on Mar 21, 2016

e2d2eeeabaf85a2c9314b8e67d1860e9ab4b5686  on Aug 24, 2019
