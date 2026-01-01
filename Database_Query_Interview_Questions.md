# Database Query Interview Questions

## 1. SQL Query Fundamentals
**Concept:** Essential SQL queries for database operations.

### Basic CRUD Operations

**1. SELECT Queries:**
```sql
-- Select all columns
SELECT * FROM Users;

-- Select specific columns
SELECT UserId, UserName, Email FROM Users;

-- With WHERE clause
SELECT * FROM Users WHERE IsActive = 1;

-- With ORDER BY
SELECT * FROM Products ORDER BY Price DESC;

-- With LIMIT/TOP
SELECT TOP 10 * FROM Orders ORDER BY OrderDate DESC;  -- SQL Server
SELECT * FROM Orders ORDER BY OrderDate DESC LIMIT 10;  -- MySQL/PostgreSQL
```

**2. INSERT Queries:**
```sql
-- Single row insert
INSERT INTO Users (UserName, Email, CreatedDate) 
VALUES ('John Doe', 'john@example.com', GETDATE());

-- Multiple rows insert
INSERT INTO Users (UserName, Email) 
VALUES 
  ('Alice', 'alice@example.com'),
  ('Bob', 'bob@example.com'),
  ('Charlie', 'charlie@example.com');

-- Insert from SELECT
INSERT INTO ArchivedOrders 
SELECT * FROM Orders WHERE OrderDate < '2023-01-01';
```

**3. UPDATE Queries:**
```sql
-- Update single column
UPDATE Users SET IsActive = 1 WHERE UserId = 123;

-- Update multiple columns
UPDATE Products 
SET Price = Price * 1.10, UpdatedDate = GETDATE() 
WHERE Category = 'Electronics';

-- Update with JOIN
UPDATE Orders 
SET Status = 'Cancelled'
FROM Orders o
INNER JOIN Users u ON o.UserId = u.UserId
WHERE u.IsActive = 0;
```

**4. DELETE Queries:**
```sql
-- Delete specific rows
DELETE FROM Users WHERE IsActive = 0;

-- Delete with JOIN
DELETE o
FROM Orders o
INNER JOIN Users u ON o.UserId = u.UserId
WHERE u.DeletedDate IS NOT NULL;

-- Truncate (faster, cannot rollback)
TRUNCATE TABLE TempLogs;
```

---

## 2. JOINs in SQL
**Concept:** Combining data from multiple tables based on related columns.

### Types of JOINs:

**1. INNER JOIN** - Returns matching records from both tables
```sql
SELECT 
  u.UserName,
  o.OrderId,
  o.OrderDate,
  o.TotalAmount
FROM Users u
INNER JOIN Orders o ON u.UserId = o.UserId
WHERE u.IsActive = 1;
```

**2. LEFT JOIN (LEFT OUTER JOIN)** - Returns all records from left table + matching from right
```sql
-- Get all users with their orders (including users without orders)
SELECT 
  u.UserName,
  o.OrderId,
  COALESCE(o.TotalAmount, 0) AS OrderAmount
FROM Users u
LEFT JOIN Orders o ON u.UserId = o.UserId;

-- Find users with NO orders
SELECT u.UserId, u.UserName
FROM Users u
LEFT JOIN Orders o ON u.UserId = o.UserId
WHERE o.OrderId IS NULL;
```

**3. RIGHT JOIN (RIGHT OUTER JOIN)** - Returns all records from right table + matching from left
```sql
SELECT 
  o.OrderId,
  u.UserName
FROM Users u
RIGHT JOIN Orders o ON u.UserId = o.UserId;
```

**4. FULL OUTER JOIN** - Returns all records from both tables
```sql
SELECT 
  COALESCE(u.UserId, o.UserId) AS UserId,
  u.UserName,
  o.OrderId
FROM Users u
FULL OUTER JOIN Orders o ON u.UserId = o.UserId;
```

**5. CROSS JOIN** - Cartesian product (all combinations)
```sql
-- Get all combinations of colors and sizes
SELECT 
  c.ColorName,
  s.SizeName
FROM Colors c
CROSS JOIN Sizes s;
```

**6. SELF JOIN** - Join table with itself
```sql
-- Find employees and their managers
SELECT 
  e.EmployeeName AS Employee,
  m.EmployeeName AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;
```

---

## 3. Aggregate Functions & GROUP BY
**Concept:** Summarizing data using aggregate functions.

### Common Aggregate Functions:

```sql
-- COUNT
SELECT COUNT(*) AS TotalUsers FROM Users;
SELECT COUNT(DISTINCT Email) AS UniqueEmails FROM Users;

-- SUM
SELECT SUM(TotalAmount) AS TotalRevenue FROM Orders;

-- AVG
SELECT AVG(Price) AS AveragePrice FROM Products;

-- MIN/MAX
SELECT 
  MIN(Price) AS LowestPrice,
  MAX(Price) AS HighestPrice
FROM Products;

-- GROUP BY
SELECT 
  Category,
  COUNT(*) AS ProductCount,
  AVG(Price) AS AvgPrice,
  SUM(Stock) AS TotalStock
FROM Products
GROUP BY Category;

-- GROUP BY with multiple columns
SELECT 
  Category,
  Brand,
  COUNT(*) AS ProductCount
FROM Products
GROUP BY Category, Brand
ORDER BY Category, Brand;

-- HAVING (filter after GROUP BY)
SELECT 
  UserId,
  COUNT(*) AS OrderCount,
  SUM(TotalAmount) AS TotalSpent
FROM Orders
GROUP BY UserId
HAVING COUNT(*) > 5  -- Users with more than 5 orders
ORDER BY TotalSpent DESC;
```

---

## 4. Subqueries
**Concept:** A query nested inside another query.

### Types of Subqueries:

**1. Scalar Subquery** (returns single value)
```sql
SELECT 
  ProductName,
  Price,
  (SELECT AVG(Price) FROM Products) AS AvgPrice
FROM Products
WHERE Price > (SELECT AVG(Price) FROM Products);
```

**2. Row Subquery** (returns single row)
```sql
SELECT * FROM Products
WHERE (Price, Stock) = (
  SELECT MAX(Price), MIN(Stock) FROM Products
);
```

**3. Column Subquery** (returns multiple values in one column)
```sql
-- Find users who placed orders
SELECT * FROM Users
WHERE UserId IN (SELECT DISTINCT UserId FROM Orders);

-- Find users who have NOT placed orders
SELECT * FROM Users
WHERE UserId NOT IN (SELECT DISTINCT UserId FROM Orders WHERE UserId IS NOT NULL);
```

**4. Table Subquery** (returns multiple rows and columns)
```sql
SELECT 
  OrderSummary.UserId,
  OrderSummary.OrderCount,
  Users.UserName
FROM (
  SELECT UserId, COUNT(*) AS OrderCount
  FROM Orders
  GROUP BY UserId
) AS OrderSummary
INNER JOIN Users ON OrderSummary.UserId = Users.UserId
WHERE OrderSummary.OrderCount > 10;
```

**5. Correlated Subquery** (references outer query)
```sql
-- Find products more expensive than category average
SELECT 
  p1.ProductName,
  p1.Category,
  p1.Price
FROM Products p1
WHERE p1.Price > (
  SELECT AVG(p2.Price)
  FROM Products p2
  WHERE p2.Category = p1.Category
);
```

---

## 5. Window Functions
**Concept:** Perform calculations across a set of rows related to the current row.

### Common Window Functions:

**1. ROW_NUMBER()** - Assigns unique sequential number
```sql
SELECT 
  UserName,
  OrderDate,
  TotalAmount,
  ROW_NUMBER() OVER (PARTITION BY UserId ORDER BY OrderDate DESC) AS RowNum
FROM Orders;

-- Get latest order for each user
WITH RankedOrders AS (
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY UserId ORDER BY OrderDate DESC) AS RowNum
  FROM Orders
)
SELECT * FROM RankedOrders WHERE RowNum = 1;
```

**2. RANK() and DENSE_RANK()**
```sql
SELECT 
  ProductName,
  Price,
  RANK() OVER (ORDER BY Price DESC) AS Rank,
  DENSE_RANK() OVER (ORDER BY Price DESC) AS DenseRank
FROM Products;
```

**3. NTILE()** - Divide rows into N groups
```sql
-- Divide customers into 4 quartiles based on spending
SELECT 
  UserId,
  TotalSpent,
  NTILE(4) OVER (ORDER BY TotalSpent DESC) AS Quartile
FROM (
  SELECT UserId, SUM(TotalAmount) AS TotalSpent
  FROM Orders
  GROUP BY UserId
) AS CustomerSpending;
```

**4. LAG() and LEAD()** - Access previous/next row
```sql
-- Compare each month's sales with previous month
SELECT 
  SaleMonth,
  Revenue,
  LAG(Revenue, 1) OVER (ORDER BY SaleMonth) AS PreviousMonth,
  Revenue - LAG(Revenue, 1) OVER (ORDER BY SaleMonth) AS GrowthAmount
FROM MonthlySales;
```

**5. Running Totals**
```sql
SELECT 
  OrderDate,
  TotalAmount,
  SUM(TotalAmount) OVER (ORDER BY OrderDate) AS RunningTotal
FROM Orders;
```

---

## 6. Common Table Expressions (CTE)
**Concept:** Temporary named result set that exists within a single query.

**Basic CTE:**
```sql
WITH ActiveUsers AS (
  SELECT UserId, UserName, Email
  FROM Users
  WHERE IsActive = 1
)
SELECT * FROM ActiveUsers WHERE Email LIKE '%@gmail.com';
```

**Multiple CTEs:**
```sql
WITH 
  ActiveUsers AS (
    SELECT UserId, UserName FROM Users WHERE IsActive = 1
  ),
  RecentOrders AS (
    SELECT UserId, OrderId, TotalAmount 
    FROM Orders 
    WHERE OrderDate >= DATEADD(DAY, -30, GETDATE())
  )
SELECT 
  u.UserName,
  COUNT(o.OrderId) AS OrderCount,
  SUM(o.TotalAmount) AS TotalSpent
FROM ActiveUsers u
LEFT JOIN RecentOrders o ON u.UserId = o.UserId
GROUP BY u.UserName;
```

**Recursive CTE** (for hierarchical data):
```sql
-- Employee hierarchy
WITH EmployeeHierarchy AS (
  -- Anchor member: top-level employees
  SELECT EmployeeId, EmployeeName, ManagerId, 1 AS Level
  FROM Employees
  WHERE ManagerId IS NULL
  
  UNION ALL
  
  -- Recursive member: employees reporting to previous level
  SELECT e.EmployeeId, e.EmployeeName, e.ManagerId, eh.Level + 1
  FROM Employees e
  INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.EmployeeId
)
SELECT * FROM EmployeeHierarchy ORDER BY Level, EmployeeName;
```

---

## 7. Indexes and Performance Optimization
**Concept:** Improving query performance using indexes.

### Types of Indexes:

**1. Clustered Index** (one per table, determines physical order)
```sql
-- Create clustered index on primary key
CREATE CLUSTERED INDEX IX_Users_UserId ON Users(UserId);
```

**2. Non-Clustered Index** (multiple allowed, separate structure)
```sql
-- Create index on frequently queried column
CREATE NONCLUSTERED INDEX IX_Users_Email ON Users(Email);

-- Composite index (multiple columns)
CREATE NONCLUSTERED INDEX IX_Orders_UserId_OrderDate 
ON Orders(UserId, OrderDate DESC);

-- Include additional columns (covering index)
CREATE NONCLUSTERED INDEX IX_Orders_UserId 
ON Orders(UserId) 
INCLUDE (OrderDate, TotalAmount);
```

**3. Unique Index**
```sql
CREATE UNIQUE INDEX IX_Users_Email_Unique ON Users(Email);
```

### Query Optimization Tips:

```sql
-- ❌ Bad: SELECT *
SELECT * FROM Users;  -- Retrieves all columns

-- ✅ Good: Select only needed columns
SELECT UserId, UserName, Email FROM Users;

-- ❌ Bad: Function on indexed column
SELECT * FROM Users WHERE UPPER(Email) = 'JOHN@EXAMPLE.COM';

-- ✅ Good: Use index directly
SELECT * FROM Users WHERE Email = 'john@example.com';

-- ❌ Bad: Leading wildcard prevents index use
SELECT * FROM Users WHERE Email LIKE '%@gmail.com';

-- ✅ Good: No leading wildcard
SELECT * FROM Users WHERE Email LIKE 'john%';

-- Use EXISTS instead of IN for large datasets
-- ❌ Slower
SELECT * FROM Users WHERE UserId IN (SELECT UserId FROM LargeTable);

-- ✅ Faster
SELECT * FROM Users u WHERE EXISTS (SELECT 1 FROM LargeTable l WHERE l.UserId = u.UserId);
```

---

## 8. Transactions & ACID Properties
**Concept:** Ensuring data integrity through transactions.

### ACID Properties:
- **Atomicity**: All or nothing
- **Consistency**: Data integrity maintained
- **Isolation**: Transactions don't interfere
- **Durability**: Changes are permanent

**Example:**
```sql
BEGIN TRANSACTION;

BEGIN TRY
  -- Deduct amount from sender
  UPDATE Accounts 
  SET Balance = Balance - 1000 
  WHERE AccountId = 123;
  
  -- Add amount to receiver
  UPDATE Accounts 
  SET Balance = Balance + 1000 
  WHERE AccountId = 456;
  
  -- Record transaction
  INSERT INTO Transactions (FromAccount, ToAccount, Amount, TransactionDate)
  VALUES (123, 456, 1000, GETDATE());
  
  COMMIT TRANSACTION;
  
END TRY
BEGIN CATCH
  ROLLBACK TRANSACTION;
  THROW;
END CATCH;
```

### Isolation Levels:
```sql
-- Read uncommitted (allows dirty reads)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Read committed (default in SQL Server)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Repeatable read
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Serializable (highest isolation)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

## 9. Real-World Query Scenarios

### Scenario 1: Find Nth Highest Salary
```sql
-- Using DENSE_RANK()
WITH RankedSalaries AS (
  SELECT 
    EmployeeName,
    Salary,
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS Rank
  FROM Employees
)
SELECT * FROM RankedSalaries WHERE Rank = 2;  -- 2nd highest

-- Using OFFSET (SQL Server 2012+, PostgreSQL)
SELECT DISTINCT Salary
FROM Employees
ORDER BY Salary DESC
OFFSET 1 ROWS FETCH NEXT 1 ROWS ONLY;  -- 2nd highest
```

### Scenario 2: Find Duplicate Records
```sql
-- Find duplicate emails
SELECT Email, COUNT(*) AS Count
FROM Users
GROUP BY Email
HAVING COUNT(*) > 1;

-- Get all duplicate records with details
SELECT *
FROM Users
WHERE Email IN (
  SELECT Email
  FROM Users
  GROUP BY Email
  HAVING COUNT(*) > 1
);
```

### Scenario 3: Delete Duplicate Records (Keep One)
```sql
-- Using CTE and ROW_NUMBER()
WITH DuplicateCTE AS (
  SELECT 
    *,
    ROW_NUMBER() OVER (PARTITION BY Email ORDER BY UserId) AS RowNum
  FROM Users
)
DELETE FROM DuplicateCTE WHERE RowNum > 1;
```

### Scenario 4: Running Total and Moving Average
```sql
SELECT 
  OrderDate,
  DailySales,
  SUM(DailySales) OVER (ORDER BY OrderDate) AS RunningTotal,
  AVG(DailySales) OVER (
    ORDER BY OrderDate 
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS SevenDayMovingAvg
FROM DailySalesReport;
```

### Scenario 5: Pivot Table (Rows to Columns)
```sql
-- Dynamic pivot example
SELECT 
  ProductName,
  [Jan], [Feb], [Mar], [Apr]
FROM (
  SELECT ProductName, SaleMonth, Revenue
  FROM Sales
) AS SourceData
PIVOT (
  SUM(Revenue)
  FOR SaleMonth IN ([Jan], [Feb], [Mar], [Apr])
) AS PivotTable;
```

---

## Summary

| Topic | Key Takeaway |
|-------|-------------|
| **CRUD Operations** | Basic INSERT, SELECT, UPDATE, DELETE queries |
| **JOINs** | INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF |
| **Aggregations** | COUNT, SUM, AVG, MIN, MAX with GROUP BY |
| **Subqueries** | Scalar, row, column, table, correlated |
| **Window Functions** | ROW_NUMBER, RANK, LAG, LEAD, running totals |
| **CTEs** | Simplify complex queries, enable recursion |
| **Indexes** | Improve performance on frequently queried columns |
| **Transactions** | ACID properties, isolation levels |
| **Optimization** | Select specific columns, use indexes, avoid functions on indexed columns |
