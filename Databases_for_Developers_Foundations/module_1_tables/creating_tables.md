- ## Creating a Table
    - To create a table, you need to define three things:

        -   Its name
        -   Its columns
        -   The data types of these columns
    - example:
        ```
        create table toys (
        toy_name varchar2(100),
        weight   number
        );
        ```
- ## Viewing Table Information
    - The data dictionary stores information about your database. You can query this to see which tables it contains. There are three key views with this information:

        - user_tables - all tables owned by the current database user
        -   all_tables - all tables your database user has access to
        - dba_tables - all tables in the database. Only available if you have DBA privileges
        ```
        select table_name, iot_name, iot_type, external, 
       partitioned, temporary, cluster_name
        from   user_tables;
        ```
- ## Try It!

Complete the following statement to create a table to store the following details about bricks:

- Colour<br>
- Shape<br>

Use the data type varchar2(10) for both columns.
When you create the table, the query should return the value "BRICKS".


```
create table bricks (
  Colour varchar2(10),
  Shape varchar2(10)
);

select table_name
from   user_tables
where  table_name = 'BRICKS';
```

- ## Table Organization

Create table in Oracle Database has an organization clause. This defines how it physically stores rows in the table.

The options for this are:

    - Heap
    - Index
    - External
By default, tables are `heap-organized`. This means the database is free to store rows wherever there is space. You can add the "organization heap" clause if you want to be explicit:
```
create table toys_heap (
  toy_name varchar2(100)
) organization heap;

select table_name, iot_name, iot_type, external, 
       partitioned, temporary, cluster_name
from   user_tables
where  table_name = 'TOYS_HEAP';
```
- ## Index-Organized Tables
Unlike a heap table, an index-organized table (IOT) imposes order on the rows within it. It physically stores rows sorted by its primary key. To create an IOT, you need to:

Specify a primary key for the table
Add the `organization index` clause at the end
For example:
```
create table toys_iot (
  toy_id   integer primary key,
  toy_name varchar2(100)
) organization index;
```
You can find IOT in the data dictionary by looking at the column IOT_TYPE. This will return IOT if the table is index-organized:
```
select table_name, iot_type
from   user_tables
where  table_name = 'TOYS_IOT';
```
- ## Try It!

Complete the following statement to create the index-organized table bricks_iot:
```
create table bricks_iot (
  bricks_id integer primary key
) /*TODO*/;

select table_name, iot_type
from   user_tables
where  table_name = 'BRICKS_IOT';
```
The query afterwards should return the following row:
```
TABLE_NAME   IOT_TYPE   
BRICKS_IOT   IOT    
```
- solution:
```
create table bricks_iot (
  bricks_id integer primary key
) organization index;

select table_name, iot_type
from   user_tables
where  table_name = 'BRICKS_IOT';
```
- ## External Tables

You use external tables to read non-database files on the database server. For example, comma-separated values (CSV) files. To do this, you need to:

Create a directory pointing to the location of the file on the server
Use the organization external clause
State the directory and name of the file you want to read
For example:
```
create or replace directory toy_dir as '/path/to/file';

create table toys_ext (
  toy_name varchar2(100)
) organization external (
  default directory tmp
  location ('toys.csv')
);
```
Note: LiveSQL doesn't support external tables. So these statements will fail!

When you query this table, it will read from the file:
```
/path/to/file/toys.csv
```
This file must be accessible to the database server. You cannot use external tables to read files on your machine!

- ## Temporary Tables
Temporary tables store session specific data. Only the session that adds the rows can see them. This can be handy to store working data.

There are two types of temporary table in Oracle Database: 

    - global and 
    - private.

- Global Temporary Tables
    - To create a global temporary table add the clause "global temporary" between create and table. For example:
```
create global temporary table toys_gtt (
  toy_name varchar2(100)
);
```

The `definition of the temporary table is permanent`. All users of the database can access it. But `only your session can view rows you insert`.

- Private Temporary Tables
    - Starting in Oracle Database 18c, you can create private temporary tables. These tables are only visible in your session. Other sessions can't see the table!

To create one use "private temporary" between create and table. You must also prefix the table name with ora$ptt_:

create private temporary table ora$ptt_toys (
  toy_name varchar2(100)
);
For both temporary table types, by default the `rows disappear when you end your transaction. You can change this to when your session ends with the "on commit" clause.

But either way, no one else can view the rows. Ensure you copy data you need to permanent tables before your session ends!

Viewing Temporary Table Details
The column temporary in the *_tables views tell you which tables are temporary:
```
select table_name, temporary
from   user_tables
where  table_name in ( 'TOYS_GTT', 'ORA$PTT_TOYS' );
Note that you can only see a row for the global temporary table. The database doesn't write private temporary tables to the data dictionary!