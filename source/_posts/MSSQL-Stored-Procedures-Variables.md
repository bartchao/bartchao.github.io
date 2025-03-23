---
title: MSSQL Stored Procedures Variables
date: 2025-03-23 13:57:35
tags: 
    - MSSQL
    - Stored Procedure
categories: 
    - Technical Notes
excerpt: Variables declaration in MSSQL
---

## View dependencies of a stored procedure

```sql=
-- Stored procedure that Microsoft will eventually remove.
EXECUTE sp_depends @objname = 'sales.GenerateSalesReport';
GO
```

```sql=
-- Objects which your stored procedure depends on.
SELECT
    [referenced_schema_name] AS 'SchemaName',
    [referenced_entity_name] AS 'TableName',
    [referenced_minor_name] AS 'ColumnName'
FROM [sys].[dm_sql_referenced_entities]('Sales.GenerateSalesReport', 'OBJECT');
GO
```

## View dependencies of a table

```sql=
-- Find out which stored procedures reference a table.
SELECT
    [referencing_schema_name],
    [referencing_entity_name],
    [referencing_id],
    [referencing_class_desc]
FROM [sys].[dm_sql_referencing_entities]('Sales.SalesPerson', 'OBJECT');
GO
```

## Best practices for writing good stored procedures

- SET NOCOUNT ON for reducing network traffic.
- Avoid using the SP\_ prefix for stored procedures.
- Always include a schema name (like dbo.) when referencing objects; without it, SQL Server checks the master database first.
- Avoid using cursors as SQL Server is optimized for set-based operations.
- Stored procedure naming convention: use verb-noun format (like Insert_Employee)

## Temp Variables vs Table Variables

### Temp Variables

```sql=
DROP TABLE IF EXISTS #SalesOrderTemp;
GO
CREATE TABLE #SalesOrderTemp
    (SalesAmount decimal(36,2), Id int);
GO
```

- Hold a subset of data
- Add indexes for performance but doesn't needed in each time.
- Lifecycle: this session is longer existed, or manually dropped.
- Global temp tables starts with two pound signs `##`
- Try matching the data type of underlining table.

## Table Variables

```sql=
DECLARE @SalesOrderVar AS TABLE
    (SalesAmount decinmal(36,2), Id int);
GO
```

Unlike temporary tables, table variables do not generate statistics, making them unsuitable for handling large amounts of data.

- Lifecycle: No need to drop since they are removed after the batch

- Unable to modify once created, cannot add column after it created.

- Table variable must declared first.

## Table-Valued Parameters

```sql=
CREATE TYPE SalesTableType
AS TABLE (SalesPersonEmail nvarchar(500));
GO
CREATE OR ALTER PROCEDURE Sales.SalesPersonReport
@SalesPersonEmail SalesTableType READONLY
```
- Pass multiple values to a stored procedure
- Table type is required for table-valued parameters
- Define the structure and insert data
- Marked as read-only and caanot be modified
- SQL doesn't create statistics on table-valued parameters

## References
[Capturing Logic with Stored Procedures in T-SQL on Pluralsight](https://app.pluralsight.com/library/courses/capturing-logic-stored-procedures-tsql/table-of-contents)