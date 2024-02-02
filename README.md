# sql_class
for the sql class

# create db
```sql
CREATE DATABASE sql_class
ON
( NAME = 'sql_class_data',
  FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL15.FORMATION\MSSQL\DATA\sql_class_data.mdf',
  SIZE = 100MB,
  MAXSIZE = UNLIMITED,
  FILEGROWTH = 10MB
)
LOG ON
( NAME = 'sql_class_log',
  FILENAME = 'C:\Program Files\Microsoft SQL Server\MSSQL15.FORMATION\MSSQL\DATA\sql_class_log.ldf',
  SIZE = 50MB,
  MAXSIZE = 100MB,
  FILEGROWTH = 5MB
)
COLLATE Latin1_General_CI_AS; -- Optional collation setting


```
# get info db
```sql
SELECT 

    mdf.database_id, 

    mdf.name, 

    mdf.physical_name as data_file, 

    ldf.physical_name as log_file, 

    db_size = CAST((mdf.size * 8.0)/1024 AS DECIMAL(8,2)), 

    log_size = CAST((ldf.size * 8.0 / 1024) AS DECIMAL(8,2))

FROM (SELECT * FROM sys.master_files WHERE type_desc = 'ROWS' ) mdf

JOIN (SELECT * FROM sys.master_files WHERE type_desc = 'LOG' ) ldf

ON mdf.database_id = ldf.database_id


```

# Create table with constrain fk and drop

```sql
CREATE TABLE department (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50),
    manager_id INT,
    location VARCHAR(50)
);
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    birthdate DATE,
    hire_date DATE,
    department_id INT, -- FK to department table
    salary DECIMAL(10, 2),
    FOREIGN KEY (department_id) REFERENCES department(department_id)
);

ALTER TABLE employees
DROP CONSTRAINT FK_employees_department;
```
# get infor db roler and user
```sql
USE sql_class;
SELECT name, type_desc 
FROM sys.database_principals 
WHERE type_desc IN ('SQL_USER', 'WINDOWS_USER');

```

# Create Role
```sql
CREATE ROLE RL_manager
GRANT SELECT ON [dbo].[employees] TO [RL_manager];
```
- First create every single object that you need in your database, and then the control.

# Trigger
```sql
CREATE TRIGGER AuditLogTrigger
ON dbo.Employee
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    DECLARE @EventTime DATETIME;
    DECLARE @TableName NVARCHAR(50);
    DECLARE @Action NVARCHAR(50);

    SET @EventTime = GETDATE();
    SET @TableName = 'Employee';

    IF EXISTS (SELECT * FROM inserted) -- can be hard to detect the action

    BEGIN
        IF EXISTS (SELECT * FROM deleted)
            SET @Action = 'Update';
        ELSE
            SET @Action = 'Insert';
    END
    ELSE
        SET @Action = 'Delete';

    INSERT INTO AuditLog (EventTime, TableName, Action)
    VALUES (@EventTime, @TableName, @Action);
END;

```

- add role and user information, time duration into audit log 
- audit log table are not allowed to be connected.
- For group database object, we should design a audit log table for our audit work.
- Based on this information, we can get the muster of each user, when they access to our data base, for example this is very usefull to find unusual activity, the security.


# Get information of roles and user from a database
```sql
--sys.database_principals
--sys.database_role_members
/*
with db_principals_info as(
select [name], [principal_id], [type_desc] from sys.database_principals
)
select * from db_principals_info
*/

--select * from sys.database_role_members
with db_principals_info as(
select [name], [principal_id], [type_desc] from sys.database_principals
)
select dp.name as username,
r.name as role 
from db_principals_info dp
left join sys.database_role_members drm
on dp.principal_id = drm.member_principal_id
left join sys.database_principals r on 
drm.role_principal_id = r.principal_id
where dp.type_desc = 'SQL_USER'
--select member_principal_id from sys.database_role_members

```
