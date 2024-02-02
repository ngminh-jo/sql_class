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

# Object of datata base

```sql
select distinct type_desc from  sys.objects
```
These objects are part of the database's metadata and serve different purposes within the database management system. Here's an explanation of each object type:

1. **SYSTEM_TABLE**: These are system catalog tables that store metadata about the database, tables, columns, indexes, and other objects in the database. They are used internally by the database management system.

2. **DEFAULT_CONSTRAINT**: A default constraint is a rule that defines a default value for a column in a table. It specifies what value should be inserted into the column if no value is provided during an INSERT operation.

3. **SQL_STORED_PROCEDURE**: A stored procedure is a set of SQL statements that can be executed as a single unit. Stored procedures are often used to encapsulate and organize complex logic in the database.

4. **FOREIGN_KEY_CONSTRAINT**: A foreign key constraint defines a relationship between two tables. It enforces referential integrity by ensuring that values in a column of one table correspond to values in a column of another table.

5. **SERVICE_QUEUE**: Service queues are used in SQL Server for message processing within Service Broker, a messaging framework for asynchronous communication between database components.

6. **USER_TABLE**: These are user-created tables used to store data in the database. They are typically the tables where application data is stored.

7. **PRIMARY_KEY_CONSTRAINT**: A primary key constraint ensures that each row in a table has a unique identifier (the primary key) and enforces data integrity by preventing duplicate values.

8. **INTERNAL_TABLE**: Internal tables are typically used by the SQL Server engine for temporary storage and query optimization purposes. They are not directly accessible or visible to users.

9. **SQL_TRIGGER**: A trigger is a database object that automatically executes a specified set of SQL statements or a user-defined function in response to a specific event, such as an INSERT, UPDATE, or DELETE operation.

10. **SQL_SCALAR_FUNCTION**: Scalar functions are user-defined functions that return a single value based on input parameters. They can be used within SQL queries and expressions.

11. **UNIQUE_CONSTRAINT**: A unique constraint ensures that values in a column or combination of columns are unique within a table, preventing duplicate values.

These object types and constraints are fundamental components of a SQL Server database, and they serve different purposes to maintain data integrity, enforce rules, and manage data within the database.

### Examples
1. **SYSTEM_TABLE**:
   - Example: `sys.tables` or `sys.columns` are system catalog tables that store information about user-created tables and columns within a database.

2. **DEFAULT_CONSTRAINT**:
   - Example: You have a "Customers" table with a "RegistrationDate" column, and you set a default constraint to insert the current date if no value is provided during an INSERT operation.

3. **SQL_STORED_PROCEDURE**:
   - Example:
     ```sql
     CREATE PROCEDURE GetEmployeeList
     AS
     BEGIN
         SELECT * FROM Employees;
     END;
     ```

4. **FOREIGN_KEY_CONSTRAINT**:
   - Example: You have an "Orders" table with a "CustomerID" column that references the "Customers" table's "CustomerID" column to ensure that each order has a valid customer.

5. **SERVICE_QUEUE**:
   - Example: Service queues are typically used within the Service Broker framework for asynchronous message processing. Detailed examples would involve Service Broker configurations and activation procedures.

6. **USER_TABLE**:
   - Example:
     ```sql
     CREATE TABLE Customers (
         CustomerID INT PRIMARY KEY,
         FirstName VARCHAR(50),
         LastName VARCHAR(50)
     );
     ```

7. **PRIMARY_KEY_CONSTRAINT**:
   - Example: In the "Products" table, you set the "ProductID" column as the primary key to ensure each product has a unique identifier.
     
8. **INTERNAL_TABLE**:
   - Internal tables are managed by the SQL Server engine and are not directly accessible by users. Examples of these tables include temporary tables created during query processing.

9. **SQL_TRIGGER**:
   - Example:
     ```sql
     CREATE TRIGGER EmployeeAudit
     ON Employees
     AFTER INSERT, UPDATE, DELETE
     AS
     BEGIN
         -- Trigger logic to log changes
     END;
     ```

10. **SQL_SCALAR_FUNCTION**:
    - Example:
      ```sql
      CREATE FUNCTION CalculateTax(@Income DECIMAL(10, 2))
      RETURNS DECIMAL(10, 2)
      AS
      BEGIN
          RETURN @Income * 0.15; -- Assuming a simple tax calculation
      END;

      declare @x_1 int
      declare @x_2 decimal(10,2) 
      set @x_1 = 5
      set @x_2 = 1.2
      select dbo.tem_func(@X_1,@x_2) 
      ```

11. **UNIQUE_CONSTRAINT**:
    - Example: In the "Emails" table, you set a unique constraint on the "EmailAddress" column to ensure that no two records can have the same email address.
