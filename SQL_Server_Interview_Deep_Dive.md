# SQL SERVER & DATABASE - KIẾN THỨC CHUYÊN SÂU CHO PHỎNG VẤN

## MỤC LỤC
1. [SQL Server - Kiến thức cơ bản](#sql-server-cơ-bản)
2. [SQL Server - Kiến thức nâng cao](#sql-server-nâng-cao)
3. [So sánh SQL Server vs MySQL vs PostgreSQL](#so-sánh-databases)
4. [Performance Tuning & Optimization](#performance-optimization)
5. [Security & Best Practices](#security-best-practices)
6. [Câu hỏi phỏng vấn thực tế](#câu-hỏi-phỏng-vấn)

---

## SQL SERVER - CƠ BẢN

### 1. SQL Server là gì?

**Câu trả lời:**
SQL Server là hệ quản trị cơ sở dữ liệu quan hệ (RDBMS) của Microsoft, sử dụng ngôn ngữ T-SQL (Transact-SQL) - phiên bản mở rộng của SQL chuẩn.

**Đặc điểm chính:**
- Hỗ trợ ACID transactions đầy đủ
- Tích hợp sâu với .NET và Windows ecosystem
- Công cụ quản lý mạnh mẽ: SQL Server Management Studio (SSMS)
- Hỗ trợ clustering, replication, high availability
- Business Intelligence tools tích hợp (SSRS, SSIS, SSAS)

---

### 2. T-SQL vs SQL chuẩn - Khác biệt gì?

**Câu trả lời:**

```sql
-- T-SQL có các tính năng mở rộng:

-- 1. Variables & Control Flow
DECLARE @counter INT = 0;
WHILE @counter < 10
BEGIN
    PRINT 'Counter: ' + CAST(@counter AS VARCHAR);
    SET @counter = @counter + 1;
END

-- 2. Error Handling với TRY...CATCH
BEGIN TRY
    BEGIN TRANSACTION
        UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
        UPDATE Accounts SET Balance = Balance + 100 WHERE AccountId = 2;
    COMMIT TRANSACTION
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION
    PRINT ERROR_MESSAGE()
END CATCH

-- 3. Stored Procedures với OUTPUT parameters
CREATE PROCEDURE GetUserCount
    @DepartmentId INT,
    @UserCount INT OUTPUT
AS
BEGIN
    SELECT @UserCount = COUNT(*) 
    FROM Users 
    WHERE DepartmentId = @DepartmentId;
END
```

**So sánh:**
| Tính năng | SQL Chuẩn | T-SQL |
|-----------|-----------|-------|
| Variables | Hạn chế | Đầy đủ với DECLARE |
| Error Handling | Cơ bản | TRY...CATCH blocks |
| Control Flow | Không | IF, WHILE, CASE |
| Temp Tables | Không | #temp, ##global temp |

---

### 3. Các kiểu Index trong SQL Server

**Câu trả lời chi tiết:**

#### a) **Clustered Index**
```sql
-- Chỉ có 1 clustered index trên mỗi table
-- Sắp xếp vật lý dữ liệu trên disk
CREATE CLUSTERED INDEX IX_Users_UserId 
ON Users(UserId);

-- Primary Key mặc định tạo Clustered Index
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY CLUSTERED,
    OrderDate DATETIME,
    CustomerId INT
);
```

**Đặc điểm:**
- Quyết định thứ tự vật lý của data
- Tìm kiếm nhanh với key column
- Range queries hiệu quả
- Chỉ có 1 clustered index/table

#### b) **Non-Clustered Index**
```sql
-- Có thể có nhiều non-clustered indexes
CREATE NONCLUSTERED INDEX IX_Users_Email 
ON Users(Email);

-- Composite Index
CREATE NONCLUSTERED INDEX IX_Orders_CustomerDate 
ON Orders(CustomerId, OrderDate);

-- Covering Index (bao gồm các cột bổ sung)
CREATE NONCLUSTERED INDEX IX_Orders_Covering
ON Orders(CustomerId)
INCLUDE (OrderDate, TotalAmount);
```

**Đặc điểm:**
- Tạo cấu trúc riêng biệt trỏ đến data thực
- Có thể có 999 non-clustered indexes/table
- Tốn thêm storage
- Cải thiện SELECT nhưng làm chậm INSERT/UPDATE/DELETE

#### c) **Unique Index**
```sql
CREATE UNIQUE NONCLUSTERED INDEX IX_Users_Email_Unique
ON Users(Email);
```

#### d) **Filtered Index**
```sql
-- Index chỉ cho một phần data (SQL Server 2008+)
CREATE NONCLUSTERED INDEX IX_Orders_Active
ON Orders(OrderDate)
WHERE Status = 'Active';
```

#### e) **Columnstore Index**
```sql
-- Tối ưu cho Data Warehouse và analytical queries
CREATE NONCLUSTERED COLUMNSTORE INDEX IX_Sales_Columnstore
ON Sales(ProductId, SaleDate, Quantity, Amount);
```

---

### 4. Execution Plan - Cách đọc và tối ưu

**Câu hỏi phỏng vấn:** "Làm thế nào để phân tích và tối ưu một query chậm?"

**Câu trả lời:**

```sql
-- 1. Xem Execution Plan
SET STATISTICS TIME ON;
SET STATISTICS IO ON;

-- 2. Actual Execution Plan (Ctrl + M trong SSMS)
SELECT u.UserName, o.OrderDate, o.TotalAmount
FROM Users u
INNER JOIN Orders o ON u.UserId = o.CustomerId
WHERE u.RegisterDate > '2024-01-01'
ORDER BY o.OrderDate DESC;

-- 3. Analyze với DMVs (Dynamic Management Views)
SELECT 
    qs.execution_count,
    qs.total_elapsed_time / 1000000.0 AS total_elapsed_time_sec,
    qs.total_worker_time / 1000000.0 AS total_cpu_time_sec,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.text)
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2) + 1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_elapsed_time DESC;
```

**Các yếu tố quan trọng trong Execution Plan:**
1. **Table Scan** ❌ (Tệ) - Quét toàn bộ table
2. **Index Scan** ⚠️ (Trung bình) - Quét toàn bộ index
3. **Index Seek** ✅ (Tốt) - Tìm kiếm chính xác trong index
4. **Key Lookup** ⚠️ - Phải tra thêm data từ clustered index
5. **Sort** ⚠️ - Tốn CPU và memory
6. **Hash Match** - Thường thấy trong JOIN lớn

**Cách tối ưu:**
```sql
-- Before: Table Scan
SELECT * FROM Orders WHERE CustomerId = 123;

-- After: Thêm index
CREATE NONCLUSTERED INDEX IX_Orders_CustomerId 
ON Orders(CustomerId);

-- Now: Index Seek
SELECT * FROM Orders WHERE CustomerId = 123;
```

---

## SQL SERVER - NÂNG CAO

### 5. Transactions & Isolation Levels

**Câu hỏi:** "Giải thích các Isolation Level và khi nào dùng từng loại?"

**Câu trả lời:**

```sql
-- 1. READ UNCOMMITTED (Lowest isolation)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
BEGIN TRANSACTION
    SELECT * FROM Accounts; -- Có thể đọc dirty data
COMMIT;

-- 2. READ COMMITTED (Default trong SQL Server)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION
    SELECT * FROM Accounts; -- Chỉ đọc committed data
COMMIT;

-- 3. REPEATABLE READ
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION
    SELECT * FROM Accounts WHERE Balance > 1000;
    -- Data sẽ không thay đổi nếu SELECT lại
    WAITFOR DELAY '00:00:05';
    SELECT * FROM Accounts WHERE Balance > 1000; -- Same result
COMMIT;

-- 4. SERIALIZABLE (Highest isolation)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION
    SELECT * FROM Accounts WHERE Balance > 1000;
    -- Không có row nào thêm vào hoặc xóa khỏi range này
COMMIT;

-- 5. SNAPSHOT (SQL Server specific)
ALTER DATABASE YourDB SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION
    SELECT * FROM Accounts; -- Đọc version tại thời điểm bắt đầu transaction
COMMIT;
```

**So sánh:**
| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|----------------|------------|---------------------|--------------|-------------|
| READ UNCOMMITTED | ✅ Có thể | ✅ Có thể | ✅ Có thể | 🚀 Nhanh nhất |
| READ COMMITTED | ❌ Không | ✅ Có thể | ✅ Có thể | ⚡ Nhanh |
| REPEATABLE READ | ❌ Không | ❌ Không | ✅ Có thể | ⚠️ Trung bình |
| SERIALIZABLE | ❌ Không | ❌ Không | ❌ Không | 🐌 Chậm |
| SNAPSHOT | ❌ Không | ❌ Không | ❌ Không | ⚡ Nhanh, không lock |

**Khi nào dùng:**
- **READ UNCOMMITTED**: Reports không cần chính xác tuyệt đối
- **READ COMMITTED**: Default, phù hợp hầu hết use cases
- **REPEATABLE READ**: Financial transactions, cần consistency
- **SERIALIZABLE**: Critical operations, ít concurrent users
- **SNAPSHOT**: High concurrency, cần consistency mà không lock

---

### 6. Deadlock - Nguyên nhân và cách xử lý

**Câu hỏi phỏng vấn:** "Giải thích Deadlock và cách debug/phòng tránh?"

**Câu trả lời:**

#### Deadlock là gì?
Deadlock xảy ra khi 2 hoặc nhiều transactions chờ nhau release locks, tạo thành vòng tròn chờ đợi vô hạn.

#### Ví dụ Deadlock:
```sql
-- Session 1:
BEGIN TRANSACTION
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
    -- Đang giữ lock trên Account 1
    WAITFOR DELAY '00:00:05';
    UPDATE Accounts SET Balance = Balance + 100 WHERE AccountId = 2;
    -- Cần lock trên Account 2
COMMIT;

-- Session 2 (chạy đồng thời):
BEGIN TRANSACTION
    UPDATE Accounts SET Balance = Balance - 50 WHERE AccountId = 2;
    -- Đang giữ lock trên Account 2
    WAITFOR DELAY '00:00:05';
    UPDATE Accounts SET Balance = Balance + 50 WHERE AccountId = 1;
    -- Cần lock trên Account 1 -> DEADLOCK!
COMMIT;
```

#### Cách phát hiện Deadlock:
```sql
-- 1. Enable Trace Flags
DBCC TRACEON(1222, -1); -- Ghi deadlock info vào Error Log

-- 2. Xem Deadlock Graph trong SQL Profiler
-- Tools -> SQL Server Profiler -> Deadlock graph event

-- 3. Query System Health Session
SELECT 
    XEvent.query('(event/data/value)[1]') AS DeadlockGraph
FROM (
    SELECT XEvent.query('.') AS XEvent
    FROM (
        SELECT CAST(target_data AS XML) AS TargetData
        FROM sys.dm_xe_session_targets st
        JOIN sys.dm_xe_sessions s ON s.address = st.event_session_address
        WHERE s.name = 'system_health'
    ) AS Data
    CROSS APPLY TargetData.nodes('//RingBufferTarget/event') AS XEventData(XEvent)
) AS src
WHERE XEvent.value('(event/@name)[1]', 'varchar(4000)') = 'xml_deadlock_report';
```

#### Cách phòng tránh Deadlock:
```sql
-- 1. Truy cập tables theo cùng một thứ tự
-- BAD:
BEGIN TRANSACTION
    UPDATE TableB SET ...;
    UPDATE TableA SET ...;
COMMIT;

-- GOOD: Tất cả transactions đều truy cập A trước, B sau
BEGIN TRANSACTION
    UPDATE TableA SET ...;
    UPDATE TableB SET ...;
COMMIT;

-- 2. Giữ transactions ngắn gọn
BEGIN TRANSACTION
    -- Chỉ UPDATE cần thiết
    UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
COMMIT;
-- Xử lý logic khác bên ngoài transaction

-- 3. Sử dụng appropriate isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 4. Sử dụng NOLOCK hint (nếu chấp nhận dirty read)
SELECT * FROM Orders WITH (NOLOCK) WHERE CustomerId = 123;

-- 5. Sử dụng UPDLOCK hint để giữ lock sớm
SELECT * FROM Accounts WITH (UPDLOCK) WHERE AccountId = 1;

-- 6. Set DEADLOCK_PRIORITY
SET DEADLOCK_PRIORITY LOW; -- Transaction này sẽ bị kill trước nếu deadlock
```

---

### 7. Stored Procedures vs Functions

**Câu hỏi:** "So sánh Stored Procedures và Functions, khi nào dùng từng loại?"

**Câu trả lời:**

#### Stored Procedures:
```sql
CREATE PROCEDURE sp_TransferMoney
    @FromAccount INT,
    @ToAccount INT,
    @Amount DECIMAL(18,2),
    @Result VARCHAR(100) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRANSACTION
            -- Kiểm tra balance
            DECLARE @Balance DECIMAL(18,2);
            SELECT @Balance = Balance FROM Accounts WHERE AccountId = @FromAccount;
            
            IF @Balance < @Amount
            BEGIN
                SET @Result = 'Insufficient funds';
                ROLLBACK;
                RETURN -1;
            END
            
            -- Thực hiện chuyển
            UPDATE Accounts SET Balance = Balance - @Amount WHERE AccountId = @FromAccount;
            UPDATE Accounts SET Balance = Balance + @Amount WHERE AccountId = @ToAccount;
            
            -- Log transaction
            INSERT INTO TransactionLog (FromAccount, ToAccount, Amount, TransDate)
            VALUES (@FromAccount, @ToAccount, @Amount, GETDATE());
            
        COMMIT TRANSACTION
        SET @Result = 'Success';
        RETURN 0;
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION
        SET @Result = ERROR_MESSAGE();
        RETURN -1;
    END CATCH
END

-- Sử dụng:
DECLARE @ResultMsg VARCHAR(100);
DECLARE @ReturnValue INT;
EXEC @ReturnValue = sp_TransferMoney 
    @FromAccount = 1, 
    @ToAccount = 2, 
    @Amount = 100,
    @Result = @ResultMsg OUTPUT;
PRINT @ResultMsg;
```

#### Scalar Functions:
```sql
CREATE FUNCTION fn_CalculateInterest
(
    @Principal DECIMAL(18,2),
    @Rate DECIMAL(5,2),
    @Years INT
)
RETURNS DECIMAL(18,2)
AS
BEGIN
    DECLARE @Interest DECIMAL(18,2);
    SET @Interest = @Principal * @Rate * @Years / 100;
    RETURN @Interest;
END

-- Sử dụng:
SELECT 
    AccountId,
    Balance,
    dbo.fn_CalculateInterest(Balance, 5.5, 1) AS YearlyInterest
FROM Accounts;
```

#### Table-Valued Functions:
```sql
-- Inline Table-Valued Function (Nhanh hơn)
CREATE FUNCTION fn_GetUserOrders(@UserId INT)
RETURNS TABLE
AS
RETURN
(
    SELECT OrderId, OrderDate, TotalAmount
    FROM Orders
    WHERE CustomerId = @UserId
);

-- Multi-Statement Table-Valued Function
CREATE FUNCTION fn_GetSalesReport(@Year INT)
RETURNS @SalesTable TABLE
(
    Month INT,
    TotalSales DECIMAL(18,2),
    OrderCount INT
)
AS
BEGIN
    INSERT INTO @SalesTable
    SELECT 
        MONTH(OrderDate) AS Month,
        SUM(TotalAmount) AS TotalSales,
        COUNT(*) AS OrderCount
    FROM Orders
    WHERE YEAR(OrderDate) = @Year
    GROUP BY MONTH(OrderDate);
    
    RETURN;
END

-- Sử dụng:
SELECT * FROM dbo.fn_GetUserOrders(123);
SELECT * FROM dbo.fn_GetSalesReport(2024);
```

#### So sánh:
| Tính năng | Stored Procedure | Function |
|-----------|-----------------|----------|
| **Return value** | Multiple result sets, OUTPUT params | Single scalar hoặc table |
| **DML Operations** | ✅ INSERT, UPDATE, DELETE | ❌ Không (chỉ SELECT) |
| **Transaction** | ✅ BEGIN TRAN, COMMIT | ❌ Không |
| **Gọi từ SELECT** | ❌ Không thể | ✅ Có thể |
| **Error Handling** | ✅ TRY...CATCH | ❌ Hạn chế |
| **Performance** | Cache execution plan | Inline functions nhanh, multi-statement chậm |
| **Use Cases** | Business logic, data modification | Calculations, filtering |

**Khi nào dùng:**
- **Stored Procedure**: Complex business logic, transactions, data modifications
- **Scalar Function**: Tính toán đơn giản, format data
- **Table Function**: Dynamic filtering, reusable table queries

---

### 8. Triggers - Các loại và Use Cases

**Câu trả lời:**

```sql
-- 1. AFTER Trigger (FOR Trigger)
CREATE TRIGGER trg_AuditUserChanges
ON Users
AFTER UPDATE, DELETE
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Log deleted records
    INSERT INTO UserAuditLog (UserId, Action, OldValue, ChangedBy, ChangedDate)
    SELECT 
        d.UserId,
        'DELETE',
        d.UserName + ', ' + d.Email,
        SYSTEM_USER,
        GETDATE()
    FROM deleted d;
    
    -- Log updated records
    INSERT INTO UserAuditLog (UserId, Action, OldValue, NewValue, ChangedBy, ChangedDate)
    SELECT 
        d.UserId,
        'UPDATE',
        d.Email,
        i.Email,
        SYSTEM_USER,
        GETDATE()
    FROM deleted d
    INNER JOIN inserted i ON d.UserId = i.UserId
    WHERE d.Email <> i.Email;
END

-- 2. INSTEAD OF Trigger
CREATE TRIGGER trg_PreventUserDelete
ON Users
INSTEAD OF DELETE
AS
BEGIN
    SET NOCOUNT ON;
    
    -- Soft delete thay vì hard delete
    UPDATE Users
    SET IsDeleted = 1, DeletedDate = GETDATE()
    WHERE UserId IN (SELECT UserId FROM deleted);
    
    PRINT 'User has been soft deleted';
END

-- 3. DDL Trigger (Database level)
CREATE TRIGGER trg_PreventTableDrop
ON DATABASE
FOR DROP_TABLE
AS
BEGIN
    PRINT 'Dropping tables is not allowed!';
    ROLLBACK;
END

-- 4. Trigger với validation
CREATE TRIGGER trg_ValidateOrderAmount
ON Orders
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    
    IF EXISTS (SELECT 1 FROM inserted WHERE TotalAmount < 0)
    BEGIN
        RAISERROR('Order amount cannot be negative', 16, 1);
        ROLLBACK TRANSACTION;
        RETURN;
    END
    
    IF EXISTS (SELECT 1 FROM inserted WHERE TotalAmount > 1000000)
    BEGIN
        -- Log high-value orders
        INSERT INTO HighValueOrderLog (OrderId, Amount, LogDate)
        SELECT OrderId, TotalAmount, GETDATE()
        FROM inserted
        WHERE TotalAmount > 1000000;
    END
END

-- Disable/Enable Trigger
DISABLE TRIGGER trg_AuditUserChanges ON Users;
ENABLE TRIGGER trg_AuditUserChanges ON Users;

-- Xem all triggers
SELECT 
    t.name AS TriggerName,
    OBJECT_NAME(t.parent_id) AS TableName,
    t.is_disabled,
    t.is_instead_of_trigger
FROM sys.triggers t;
```

---

### 9. Common Table Expressions (CTEs) & Window Functions

**Câu hỏi:** "Sử dụng CTE và Window Functions để giải quyết bài toán phức tạp?"

**Câu trả lời:**

```sql
-- 1. CTE cơ bản - Lấy top 3 products theo category
WITH ProductRanking AS (
    SELECT 
        ProductId,
        ProductName,
        CategoryId,
        Price,
        ROW_NUMBER() OVER (PARTITION BY CategoryId ORDER BY Price DESC) AS PriceRank
    FROM Products
)
SELECT * FROM ProductRanking WHERE PriceRank <= 3;

-- 2. Recursive CTE - Employee Hierarchy
WITH EmployeeHierarchy AS (
    -- Anchor member: Top-level managers
    SELECT 
        EmployeeId,
        ManagerId,
        FullName,
        0 AS Level,
        CAST(FullName AS VARCHAR(1000)) AS HierarchyPath
    FROM Employees
    WHERE ManagerId IS NULL
    
    UNION ALL
    
    -- Recursive member
    SELECT 
        e.EmployeeId,
        e.ManagerId,
        e.FullName,
        eh.Level + 1,
        CAST(eh.HierarchyPath + ' -> ' + e.FullName AS VARCHAR(1000))
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.EmployeeId
)
SELECT * FROM EmployeeHierarchy
ORDER BY HierarchyPath;

-- 3. Multiple CTEs
WITH MonthlySales AS (
    SELECT 
        CustomerId,
        YEAR(OrderDate) AS SaleYear,
        MONTH(OrderDate) AS SaleMonth,
        SUM(TotalAmount) AS MonthlyTotal
    FROM Orders
    GROUP BY CustomerId, YEAR(OrderDate), MONTH(OrderDate)
),
CustomerTrend AS (
    SELECT 
        CustomerId,
        SaleYear,
        SaleMonth,
        MonthlyTotal,
        LAG(MonthlyTotal) OVER (PARTITION BY CustomerId ORDER BY SaleYear, SaleMonth) AS PrevMonthTotal,
        LEAD(MonthlyTotal) OVER (PARTITION BY CustomerId ORDER BY SaleYear, SaleMonth) AS NextMonthTotal
    FROM MonthlySales
)
SELECT 
    CustomerId,
    SaleYear,
    SaleMonth,
    MonthlyTotal,
    MonthlyTotal - PrevMonthTotal AS GrowthFromPrevMonth,
    CASE 
        WHEN MonthlyTotal > PrevMonthTotal THEN 'Growing'
        WHEN MonthlyTotal < PrevMonthTotal THEN 'Declining'
        ELSE 'Stable'
    END AS Trend
FROM CustomerTrend;

-- 4. Window Functions - Running Total
SELECT 
    OrderId,
    OrderDate,
    TotalAmount,
    SUM(TotalAmount) OVER (ORDER BY OrderDate) AS RunningTotal,
    AVG(TotalAmount) OVER (ORDER BY OrderDate ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS Moving7DayAvg
FROM Orders;

-- 5. NTILE - Chia thành quartiles
SELECT 
    ProductId,
    ProductName,
    Price,
    NTILE(4) OVER (ORDER BY Price) AS PriceQuartile
FROM Products;

-- 6. FIRST_VALUE và LAST_VALUE
SELECT 
    ProductId,
    CategoryId,
    Price,
    FIRST_VALUE(ProductName) OVER (PARTITION BY CategoryId ORDER BY Price DESC) AS MostExpensiveInCategory,
    LAST_VALUE(ProductName) OVER (
        PARTITION BY CategoryId 
        ORDER BY Price DESC 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS CheapestInCategory
FROM Products;
```

---

### 10. Performance Tuning - Chiến lược tối ưu

**Câu hỏi phỏng vấn:** "Một query chạy chậm, bạn làm gì để tối ưu?"

**Câu trả lời chi tiết:**

#### Bước 1: Phân tích hiện trạng
```sql
-- 1. Xem execution plan
SET STATISTICS TIME ON;
SET STATISTICS IO ON;

-- Your slow query here
SELECT u.UserName, COUNT(o.OrderId) AS OrderCount
FROM Users u
LEFT JOIN Orders o ON u.UserId = o.CustomerId
WHERE u.RegisterDate > '2024-01-01'
GROUP BY u.UserName
ORDER BY OrderCount DESC;

-- 2. Kiểm tra missing indexes
SELECT 
    migs.avg_total_user_cost * (migs.avg_user_impact / 100.0) * (migs.user_seeks + migs.user_scans) AS improvement_measure,
    'CREATE INDEX [IX_' + OBJECT_NAME(mid.object_id) + '_' + REPLACE(REPLACE(mid.equality_columns, '[', ''), ']', '') + ']' +
    ' ON ' + mid.statement + ' (' + ISNULL(mid.equality_columns, '') + 
    CASE WHEN mid.inequality_columns IS NOT NULL THEN ',' + mid.inequality_columns ELSE '' END + ')' +
    CASE WHEN mid.included_columns IS NOT NULL THEN ' INCLUDE (' + mid.included_columns + ')' ELSE '' END AS create_index_statement
FROM sys.dm_db_missing_index_groups mig
INNER JOIN sys.dm_db_missing_index_group_stats migs ON migs.group_handle = mig.index_group_handle
INNER JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
ORDER BY improvement_measure DESC;
```

#### Bước 2: Các kỹ thuật tối ưu

```sql
-- 1. Sử dụng appropriate indexes
-- Before: Table Scan
SELECT * FROM Orders WHERE CustomerId = 123;

-- After: Index Seek
CREATE NONCLUSTERED INDEX IX_Orders_CustomerId ON Orders(CustomerId);

-- 2. Covering Index
-- Before: Index Seek + Key Lookup
SELECT CustomerId, OrderDate, TotalAmount FROM Orders WHERE CustomerId = 123;

-- After: Index Seek only (no Key Lookup)
CREATE NONCLUSTERED INDEX IX_Orders_Covering
ON Orders(CustomerId) INCLUDE (OrderDate, TotalAmount);

-- 3. Avoid functions on indexed columns
-- BAD: Index không được sử dụng
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2024;

-- GOOD: Index được sử dụng
SELECT * FROM Orders WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01';

-- 4. Use EXISTS thay vì IN cho subquery lớn
-- BAD: Slower với large subquery
SELECT * FROM Users WHERE UserId IN (SELECT CustomerId FROM Orders WHERE OrderDate > '2024-01-01');

-- GOOD: Faster
SELECT * FROM Users u WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerId = u.UserId AND o.OrderDate > '2024-01-01');

-- 5. Paging với OFFSET-FETCH (thay vì TOP)
-- Efficient paging
SELECT OrderId, OrderDate, TotalAmount
FROM Orders
ORDER BY OrderDate DESC
OFFSET 100 ROWS FETCH NEXT 50 ROWS ONLY;

-- 6. Partition large tables
CREATE PARTITION FUNCTION pf_OrderDate (DATETIME)
AS RANGE RIGHT FOR VALUES 
    ('2023-01-01', '2024-01-01', '2025-01-01');

CREATE PARTITION SCHEME ps_OrderDate
AS PARTITION pf_OrderDate ALL TO ([PRIMARY]);

CREATE TABLE Orders (
    OrderId INT,
    OrderDate DATETIME,
    ...
) ON ps_OrderDate(OrderDate);

-- 7. Update Statistics
UPDATE STATISTICS Orders WITH FULLSCAN;

-- 8. Rebuild fragmented indexes
SELECT 
    OBJECT_NAME(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
INNER JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 30;

-- Rebuild index
ALTER INDEX IX_Orders_CustomerId ON Orders REBUILD;
```

---

## SO SÁNH: SQL SERVER vs MYSQL vs POSTGRESQL

### Bảng so sánh tổng quan:

| Tiêu chí | SQL Server | MySQL | PostgreSQL |
|----------|-----------|-------|------------|
| **Nhà phát triển** | Microsoft | Oracle | Open Source Community |
| **License** | Commercial | Open Source / Commercial | Open Source |
| **Platform** | Windows, Linux | Cross-platform | Cross-platform |
| **ACID Compliance** | ✅ Full | ✅ InnoDB engine | ✅ Full |
| **Storage Engines** | Single | Multiple (InnoDB, MyISAM) | Single (extensible) |
| **Replication** | Built-in, advanced | Built-in, mature | Built-in, logical & physical |
| **Full-Text Search** | ✅ | ✅ | ✅ |
| **JSON Support** | ✅ (2016+) | ✅ (5.7+) | ✅ Native, excellent |
| **Window Functions** | ✅ | ✅ (8.0+) | ✅ |
| **CTEs** | ✅ | ✅ (8.0+) | ✅ |
| **Stored Procedures** | T-SQL | Limited | PL/pgSQL (powerful) |
| **Performance** | Excellent | Very good | Excellent |
| **Scalability** | Enterprise-level | Good | Very good |
| **Cost** | Expensive | Free / Paid | Free |

### Syntax khác biệt:

```sql
-- 1. AUTO INCREMENT
-- SQL Server:
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    UserName VARCHAR(100)
);

-- MySQL:
CREATE TABLE Users (
    UserId INT AUTO_INCREMENT PRIMARY KEY,
    UserName VARCHAR(100)
);

-- PostgreSQL:
CREATE TABLE Users (
    UserId SERIAL PRIMARY KEY,
    UserName VARCHAR(100)
);

-- 2. TOP / LIMIT
-- SQL Server:
SELECT TOP 10 * FROM Users ORDER BY RegisterDate DESC;

-- MySQL / PostgreSQL:
SELECT * FROM Users ORDER BY RegisterDate DESC LIMIT 10;

-- SQL Server (Standard SQL):
SELECT * FROM Users ORDER BY RegisterDate DESC OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

-- 3. STRING CONCATENATION
-- SQL Server:
SELECT FirstName + ' ' + LastName AS FullName FROM Users;
SELECT CONCAT(FirstName, ' ', LastName) AS FullName FROM Users;

-- MySQL:
SELECT CONCAT(FirstName, ' ', LastName) AS FullName FROM Users;

-- PostgreSQL:
SELECT FirstName || ' ' || LastName AS FullName FROM Users;
SELECT CONCAT(FirstName, ' ', LastName) AS FullName FROM Users;

-- 4. DATE FUNCTIONS
-- SQL Server:
SELECT GETDATE(); -- Current datetime
SELECT DATEADD(day, 7, OrderDate) FROM Orders;
SELECT DATEDIFF(day, StartDate, EndDate) FROM Projects;

-- MySQL:
SELECT NOW(); -- Current datetime
SELECT DATE_ADD(OrderDate, INTERVAL 7 DAY) FROM Orders;
SELECT DATEDIFF(EndDate, StartDate) FROM Projects;

-- PostgreSQL:
SELECT NOW(); -- Current datetime
SELECT OrderDate + INTERVAL '7 days' FROM Orders;
SELECT EndDate - StartDate FROM Projects; -- Returns interval

-- 5. IF/CASE
-- SQL Server:
IF EXISTS (SELECT 1 FROM Users WHERE UserId = 1)
    PRINT 'User exists'
ELSE
    PRINT 'User not found';

-- MySQL:
SELECT IF(COUNT(*) > 0, 'User exists', 'User not found')
FROM Users WHERE UserId = 1;

-- PostgreSQL (in function):
DO $$
BEGIN
    IF EXISTS (SELECT 1 FROM Users WHERE UserId = 1) THEN
        RAISE NOTICE 'User exists';
    ELSE
        RAISE NOTICE 'User not found';
    END IF;
END $$;
```

---

## CÂU HỎI PHỎNG VẤN THỰC TÊ

### Câu 1: "Giải thích N+1 Query problem và cách giải quyết?"

**Câu trả lời:**

N+1 Query problem xảy ra khi:
1. Query đầu tiên lấy N records
2. Mỗi record trigger thêm 1 query để lấy related data
3. Tổng cộng: 1 + N queries (rất chậm với N lớn)

```sql
-- BAD: N+1 Problem
-- Query 1: Lấy 100 users
SELECT * FROM Users;

-- Query 2-101: Với mỗi user, lấy orders (100 queries)
-- Trong application code:
foreach (var user in users) {
    var orders = db.Query("SELECT * FROM Orders WHERE CustomerId = " + user.UserId);
}

-- GOOD: Single query với JOIN
SELECT 
    u.UserId, u.UserName,
    o.OrderId, o.OrderDate, o.TotalAmount
FROM Users u
LEFT JOIN Orders o ON u.UserId = o.CustomerId;

-- GOOD: Sử dụng IN clause
SELECT * FROM Users;
SELECT * FROM Orders WHERE CustomerId IN (1,2,3,4,5...); -- Single query
```

### Câu 2: "Tạo database schema cho e-commerce với best practices?"

**Câu trả lời:**

```sql
-- 1. Users Table
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    Email VARCHAR(255) NOT NULL UNIQUE,
    PasswordHash VARCHAR(255) NOT NULL,
    FirstName NVARCHAR(100),
    LastName NVARCHAR(100),
    Phone VARCHAR(20),
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    IsActive BIT DEFAULT 1,
    INDEX IX_Users_Email (Email),
    INDEX IX_Users_CreatedAt (CreatedAt)
);

-- 2. Products Table
CREATE TABLE Products (
    ProductId INT IDENTITY(1,1) PRIMARY KEY,
    ProductName NVARCHAR(255) NOT NULL,
    SKU VARCHAR(50) NOT NULL UNIQUE,
    Description NVARCHAR(MAX),
    Price DECIMAL(18,2) NOT NULL CHECK (Price >= 0),
    Stock INT NOT NULL DEFAULT 0 CHECK (Stock >= 0),
    CategoryId INT,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    IsActive BIT DEFAULT 1,
    INDEX IX_Products_CategoryId (CategoryId),
    INDEX IX_Products_SKU (SKU),
    INDEX IX_Products_Price (Price)
);

-- 3. Categories Table
CREATE TABLE Categories (
    CategoryId INT IDENTITY(1,1) PRIMARY KEY,
    CategoryName NVARCHAR(100) NOT NULL,
    ParentCategoryId INT NULL,
    Description NVARCHAR(500),
    FOREIGN KEY (ParentCategoryId) REFERENCES Categories(CategoryId)
);

-- 4. Orders Table
CREATE TABLE Orders (
    OrderId INT IDENTITY(1,1) PRIMARY KEY,
    OrderNumber VARCHAR(50) NOT NULL UNIQUE,
    UserId INT NOT NULL,
    OrderDate DATETIME2 DEFAULT GETDATE(),
    ShippingAddress NVARCHAR(500),
    TotalAmount DECIMAL(18,2) NOT NULL,
    Status VARCHAR(50) DEFAULT 'Pending',
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (UserId) REFERENCES Users(UserId),
    INDEX IX_Orders_UserId (UserId),
    INDEX IX_Orders_OrderDate (OrderDate),
    INDEX IX_Orders_Status (Status)
);

-- 5. OrderItems Table
CREATE TABLE OrderItems (
    OrderItemId INT IDENTITY(1,1) PRIMARY KEY,
    OrderId INT NOT NULL,
    ProductId INT NOT NULL,
    Quantity INT NOT NULL CHECK (Quantity > 0),
    UnitPrice DECIMAL(18,2) NOT NULL,
    Subtotal AS (Quantity * UnitPrice) PERSISTED,
    FOREIGN KEY (OrderId) REFERENCES Orders(OrderId) ON DELETE CASCADE,
    FOREIGN KEY (ProductId) REFERENCES Products(ProductId),
    INDEX IX_OrderItems_OrderId (OrderId),
    INDEX IX_OrderItems_ProductId (ProductId)
);

-- 6. Audit Trail Table
CREATE TABLE AuditLog (
    AuditId BIGINT IDENTITY(1,1) PRIMARY KEY,
    TableName VARCHAR(100) NOT NULL,
    RecordId INT NOT NULL,
    Action VARCHAR(20) NOT NULL, -- INSERT, UPDATE, DELETE
    OldValue NVARCHAR(MAX),
    NewValue NVARCHAR(MAX),
    ChangedBy VARCHAR(100) NOT NULL,
    ChangedAt DATETIME2 DEFAULT GETDATE(),
    INDEX IX_AuditLog_TableName_RecordId (TableName, RecordId),
    INDEX IX_AuditLog_ChangedAt (ChangedAt)
);

-- 7. Trigger tự động update UpdatedAt
CREATE TRIGGER trg_Users_UpdateTimestamp
ON Users
AFTER UPDATE
AS
BEGIN
    UPDATE Users
    SET UpdatedAt = GETDATE()
    WHERE UserId IN (SELECT UserId FROM inserted);
END
```

### Câu 3: "Implement soft delete và query efficiently?"

**Câu trả lời:**

```sql
-- 1. Add soft delete columns
ALTER TABLE Users ADD 
    IsDeleted BIT DEFAULT 0,
    DeletedAt DATETIME2 NULL,
    DeletedBy VARCHAR(100) NULL;

-- 2. Create filtered index (only active records)
CREATE NONCLUSTERED INDEX IX_Users_Active
ON Users(UserId, Email)
WHERE IsDeleted = 0;

-- 3. Create view for active users
CREATE VIEW vw_ActiveUsers
AS
SELECT * FROM Users WHERE IsDeleted = 0;

-- 4. Soft delete procedure
CREATE PROCEDURE sp_SoftDeleteUser
    @UserId INT,
    @DeletedBy VARCHAR(100)
AS
BEGIN
    UPDATE Users
    SET 
        IsDeleted = 1,
        DeletedAt = GETDATE(),
        DeletedBy = @DeletedBy
    WHERE UserId = @UserId;
END

-- 5. Query only active users
SELECT * FROM Users WHERE IsDeleted = 0;
-- Or
SELECT * FROM vw_ActiveUsers;

-- 6. Restore deleted user
CREATE PROCEDURE sp_RestoreUser
    @UserId INT
AS
BEGIN
    UPDATE Users
    SET 
        IsDeleted = 0,
        DeletedAt = NULL,
        DeletedBy = NULL
    WHERE UserId = @UserId;
END
```

### Câu 4: "Handle concurrent updates với Optimistic Locking?"

**Câu trả lời:**

```sql
-- 1. Add RowVersion column
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    ProductName NVARCHAR(255),
    Price DECIMAL(18,2),
    Stock INT,
    RowVersion ROWVERSION -- Tự động update mỗi lần modify
);

-- 2. Update với version check
CREATE PROCEDURE sp_UpdateProductPrice
    @ProductId INT,
    @NewPrice DECIMAL(18,2),
    @ExpectedRowVersion BINARY(8)
AS
BEGIN
    DECLARE @RowsAffected INT;
    
    UPDATE Products
    SET Price = @NewPrice
    WHERE ProductId = @ProductId 
        AND RowVersion = @ExpectedRowVersion;
    
    SET @RowsAffected = @@ROWCOUNT;
    
    IF @RowsAffected = 0
    BEGIN
        RAISERROR('Concurrency conflict: Record has been modified by another user', 16, 1);
        RETURN -1;
    END
    
    RETURN 0;
END

-- 3. Application code usage (C# example):
/*
// Step 1: Read with current version
var product = db.Query<Product>("SELECT ProductId, Price, RowVersion FROM Products WHERE ProductId = @id");

// Step 2: User modifies price
product.Price = 29.99;

// Step 3: Update with version check
var result = db.Execute("sp_UpdateProductPrice", new {
    ProductId = product.ProductId,
    NewPrice = product.Price,
    ExpectedRowVersion = product.RowVersion
});

if (result == -1) {
    // Handle concurrency conflict
    // Option 1: Reload and retry
    // Option 2: Show error to user
}
*/
```

### Câu 5: "Design và implement full-text search?"

**Câu trả lời:**

```sql
-- 1. Enable full-text search on database
CREATE FULLTEXT CATALOG ftCatalog AS DEFAULT;

-- 2. Create full-text index
CREATE FULLTEXT INDEX ON Products(ProductName, Description)
KEY INDEX PK_Products
WITH STOPLIST = SYSTEM;

-- 3. Search với CONTAINS
SELECT ProductId, ProductName, Description
FROM Products
WHERE CONTAINS((ProductName, Description), 'laptop');

-- 4. Search với FREETEXT (less strict)
SELECT ProductId, ProductName, Description
FROM Products
WHERE FREETEXT((ProductName, Description), 'gaming laptop computer');

-- 5. Ranked search results
SELECT 
    p.ProductId, 
    p.ProductName,
    ft.RANK
FROM Products p
INNER JOIN FREETEXTTABLE(Products, (ProductName, Description), 'laptop gaming') AS ft
    ON p.ProductId = ft.[KEY]
ORDER BY ft.RANK DESC;

-- 6. Advanced search với proximity
SELECT * FROM Products
WHERE CONTAINS(Description, 'NEAR((laptop, gaming), 5)'); -- Within 5 words

-- 7. Alternative: LIKE pattern (slower, but simpler)
SELECT * FROM Products
WHERE ProductName LIKE '%laptop%'
   OR Description LIKE '%laptop%';

-- 8. Alternative: Using indexes for LIKE
CREATE INDEX IX_Products_Name_Pattern 
ON Products(ProductName)
WHERE ProductName LIKE '[A-Z]%'; -- Only for prefix searches
```

---

## BEST PRACTICES & SECURITY

### 1. SQL Injection Prevention

```sql
-- ❌ NEVER DO THIS (Vulnerable to SQL Injection)
CREATE PROCEDURE sp_GetUser_Unsafe
    @Email VARCHAR(255)
AS
BEGIN
    DECLARE @SQL NVARCHAR(MAX);
    SET @SQL = 'SELECT * FROM Users WHERE Email = ''' + @Email + '''';
    EXEC sp_executesql @SQL;
END
-- Attacker có thể inject: ' OR '1'='1

-- ✅ ALWAYS DO THIS (Safe with parameters)
CREATE PROCEDURE sp_GetUser_Safe
    @Email VARCHAR(255)
AS
BEGIN
    SELECT * FROM Users WHERE Email = @Email;
END

-- ✅ Dynamic SQL với parameters
CREATE PROCEDURE sp_DynamicQuery_Safe
    @TableName NVARCHAR(128),
    @Email VARCHAR(255)
AS
BEGIN
    DECLARE @SQL NVARCHAR(MAX);
    
    -- Validate table name (whitelist approach)
    IF @TableName NOT IN ('Users', 'Customers', 'Employees')
    BEGIN
        RAISERROR('Invalid table name', 16, 1);
        RETURN;
    END
    
    SET @SQL = N'SELECT * FROM ' + QUOTENAME(@TableName) + ' WHERE Email = @EmailParam';
    EXEC sp_executesql @SQL, N'@EmailParam VARCHAR(255)', @EmailParam = @Email;
END
```

### 2. Performance Best Practices

```sql
-- 1. Always use SET NOCOUNT ON in stored procedures
CREATE PROCEDURE sp_Example
AS
BEGIN
    SET NOCOUNT ON; -- Reduces network traffic
    -- Your code here
END

-- 2. Use appropriate data types (smaller is better)
-- ❌ BAD
CREATE TABLE Orders (
    OrderId BIGINT,
    Status VARCHAR(MAX)
);

-- ✅ GOOD
CREATE TABLE Orders (
    OrderId INT, -- BIGINT only if > 2 billion records
    Status VARCHAR(20) -- Fixed length for known values
);

-- 3. Avoid SELECT *
-- ❌ BAD
SELECT * FROM Users;

-- ✅ GOOD
SELECT UserId, UserName, Email FROM Users;

-- 4. Use EXISTS instead of COUNT for existence check
-- ❌ BAD
IF (SELECT COUNT(*) FROM Users WHERE Email = @Email) > 0
    PRINT 'Exists';

-- ✅ GOOD
IF EXISTS (SELECT 1 FROM Users WHERE Email = @Email)
    PRINT 'Exists';
```

### 3. Security Best Practices

```sql
-- 1. Encrypt sensitive data
-- Enable TDE (Transparent Data Encryption)
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE MyServerCert;
ALTER DATABASE YourDB SET ENCRYPTION ON;

-- 2. Always Encrypted for column-level encryption
CREATE TABLE Users (
    UserId INT PRIMARY KEY,
    Email VARCHAR(255),
    SSN VARCHAR(11) ENCRYPTED WITH (
        ENCRYPTION_TYPE = DETERMINISTIC,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256',
        COLUMN_ENCRYPTION_KEY = CEK1
    )
);

-- 3. Use roles for access control
CREATE ROLE ReadOnlyRole;
GRANT SELECT ON dbo.Users TO ReadOnlyRole;
GRANT SELECT ON dbo.Orders TO ReadOnlyRole;

CREATE ROLE DataEntryRole;
GRANT SELECT, INSERT, UPDATE ON dbo.Orders TO DataEntryRole;

-- 4. Row-Level Security
CREATE FUNCTION fn_SecurityPredicate(@UserId INT)
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS result
WHERE @UserId = CAST(SESSION_CONTEXT(N'UserId') AS INT);

CREATE SECURITY POLICY UserAccessPolicy
ADD FILTER PREDICATE dbo.fn_SecurityPredicate(UserId) ON dbo.Orders;
```

---

## TÀI LIỆU THAM KHẢO

### Official Documentation:
- SQL Server: https://docs.microsoft.com/sql/sql-server/
- MySQL: https://dev.mysql.com/doc/
- PostgreSQL: https://www.postgresql.org/docs/

### Tools:
- **SQL Server Management Studio (SSMS)**: IDE chính thức
- **Azure Data Studio**: Cross-platform alternative
- **SQL Server Profiler**: Query performance analysis
- **Database Engine Tuning Advisor**: Index recommendations

### Interview Preparation:
- LeetCode Database: https://leetcode.com/problemset/database/
- HackerRank SQL: https://www.hackerrank.com/domains/sql
- SQLZoo: https://sqlzoo.net/

---

## CHECKLIST CHUẨN BỊ PHỎNG VẤN

✅ **Cơ bản:**
- [ ] Hiểu ACID properties
- [ ] Các loại JOINs (INNER, LEFT, RIGHT, FULL, CROSS)
- [ ] Index types (Clustered, Non-Clustered)
- [ ] Primary Key vs Unique Key vs Foreign Key
- [ ] Normalization (1NF, 2NF, 3NF, BCNF)

✅ **Trung cấp:**
- [ ] Execution plans và cách optimize
- [ ] Stored Procedures vs Functions
- [ ] Triggers và use cases
- [ ] Transactions & Isolation Levels
- [ ] CTEs và Window Functions

✅ **Nâng cao:**
- [ ] Deadlock detection và prevention
- [ ] Partitioning strategies
- [ ] Replication types
- [ ] Query optimization techniques
- [ ] Security best practices

✅ **SQL Server Specific:**
- [ ] T-SQL syntax đặc thù
- [ ] SSMS và các tools
- [ ] Integration Services (SSIS)
- [ ] Reporting Services (SSRS)
- [ ] Always On Availability Groups

---

**Chúc bạn phỏng vấn thành công! 🚀**
