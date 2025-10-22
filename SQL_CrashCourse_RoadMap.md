# SQL CRASH COURSE - LỘ TRÌNH HỌC CẤP TỐC CHO PHỎNG VẤN

## 🎯 MỤC TIÊU: Từ ZERO → PASS PHỎNG VẤN trong thời gian ngắn

---

## NGÀY 1-2: FOUNDATION - CƠ BẢN NHẤT (PHẢI BIẾT)

### 1.1. SELECT - Query cơ bản (30 phút)

```sql
-- CÚ PHÁP CƠ BẢN NHẤT
SELECT cột1, cột2 FROM tên_bảng;

-- VÍ DỤ THỰC TẾ: Bảng Users
/*
UserId | UserName  | Email              | Age | City
-------|-----------|--------------------|----|--------
1      | John      | john@email.com     | 25 | Hanoi
2      | Alice     | alice@email.com    | 30 | HCMC
3      | Bob       | bob@email.com      | 22 | Hanoi
*/

-- Query 1: Lấy tất cả users
SELECT * FROM Users;

-- Query 2: Chỉ lấy tên và email
SELECT UserName, Email FROM Users;

-- Query 3: Lọc theo điều kiện WHERE
SELECT * FROM Users WHERE City = 'Hanoi';

-- Query 4: Nhiều điều kiện
SELECT * FROM Users WHERE Age > 25 AND City = 'HCMC';
SELECT * FROM Users WHERE City = 'Hanoi' OR City = 'HCMC';
```

**🎯 BÀI TẬP THỰC HÀNH:**
```sql
-- Tạo bảng để luyện tập (copy vào SQL Server và chạy)
CREATE TABLE Users (
    UserId INT PRIMARY KEY IDENTITY(1,1),
    UserName VARCHAR(100),
    Email VARCHAR(255),
    Age INT,
    City VARCHAR(100)
);

INSERT INTO Users VALUES ('John', 'john@email.com', 25, 'Hanoi');
INSERT INTO Users VALUES ('Alice', 'alice@email.com', 30, 'HCMC');
INSERT INTO Users VALUES ('Bob', 'bob@email.com', 22, 'Hanoi');
INSERT INTO Users VALUES ('Charlie', 'charlie@email.com', 35, 'Danang');

-- BÀI TẬP: Tự làm
-- 1. Lấy tất cả users ở Hanoi
-- 2. Lấy users có tuổi từ 25 đến 35
-- 3. Lấy users không ở HCMC
```

---

### 1.2. WHERE Clause - Điều kiện lọc (30 phút)

```sql
-- Các toán tử so sánh
SELECT * FROM Users WHERE Age = 25;        -- Bằng
SELECT * FROM Users WHERE Age > 25;        -- Lớn hơn
SELECT * FROM Users WHERE Age >= 25;       -- Lớn hơn hoặc bằng
SELECT * FROM Users WHERE Age < 30;        -- Nhỏ hơn
SELECT * FROM Users WHERE Age <= 30;       -- Nhỏ hơn hoặc bằng
SELECT * FROM Users WHERE Age <> 25;       -- Khác (!=)

-- BETWEEN: Trong khoảng
SELECT * FROM Users WHERE Age BETWEEN 25 AND 35;

-- IN: Nằm trong danh sách
SELECT * FROM Users WHERE City IN ('Hanoi', 'HCMC', 'Danang');

-- LIKE: Tìm kiếm pattern
SELECT * FROM Users WHERE UserName LIKE 'J%';      -- Bắt đầu bằng J
SELECT * FROM Users WHERE UserName LIKE '%o%';     -- Chứa chữ o
SELECT * FROM Users WHERE Email LIKE '%@gmail.com'; -- Email Gmail

-- IS NULL / IS NOT NULL
SELECT * FROM Users WHERE City IS NULL;
SELECT * FROM Users WHERE City IS NOT NULL;
```

**🎯 CÂU HỎI PHỎNG VẤN THƯỜNG GẶP:**
> **"Sự khác biệt giữa WHERE và HAVING?"**
```sql
-- WHERE: Lọc TRƯỚC khi group
-- HAVING: Lọc SAU khi group

-- Ví dụ:
SELECT City, COUNT(*) AS UserCount
FROM Users
WHERE Age > 20                    -- WHERE: Lọc users trước
GROUP BY City
HAVING COUNT(*) > 1;              -- HAVING: Lọc cities sau khi đếm
```

---

### 1.3. ORDER BY & LIMIT (20 phút)

```sql
-- Sắp xếp tăng dần (ASC - mặc định)
SELECT * FROM Users ORDER BY Age;
SELECT * FROM Users ORDER BY Age ASC;

-- Sắp xếp giảm dần (DESC)
SELECT * FROM Users ORDER BY Age DESC;

-- Sắp xếp nhiều cột
SELECT * FROM Users ORDER BY City ASC, Age DESC;

-- TOP (SQL Server) / LIMIT (MySQL, PostgreSQL)
-- SQL Server:
SELECT TOP 5 * FROM Users ORDER BY Age DESC;

-- MySQL / PostgreSQL:
SELECT * FROM Users ORDER BY Age DESC LIMIT 5;

-- SQL Server (Standard SQL):
SELECT * FROM Users ORDER BY Age DESC 
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;

-- Paging (Phân trang)
SELECT * FROM Users ORDER BY UserId 
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;  -- Page 2, mỗi page 10 records
```

---

### 1.4. Aggregate Functions - Hàm tổng hợp (30 phút)

```sql
-- COUNT: Đếm số records
SELECT COUNT(*) FROM Users;                    -- Tổng số users
SELECT COUNT(*) FROM Users WHERE City = 'Hanoi'; -- Users ở Hanoi

-- SUM: Tổng
SELECT SUM(Age) FROM Users;

-- AVG: Trung bình
SELECT AVG(Age) AS AverageAge FROM Users;

-- MAX / MIN: Lớn nhất / Nhỏ nhất
SELECT MAX(Age) AS OldestAge FROM Users;
SELECT MIN(Age) AS YoungestAge FROM Users;

-- Kết hợp với GROUP BY
SELECT 
    City,
    COUNT(*) AS UserCount,
    AVG(Age) AS AvgAge,
    MAX(Age) AS MaxAge,
    MIN(Age) AS MinAge
FROM Users
GROUP BY City;

-- HAVING: Lọc sau khi GROUP BY
SELECT City, COUNT(*) AS UserCount
FROM Users
GROUP BY City
HAVING COUNT(*) > 1;  -- Chỉ lấy cities có > 1 user
```

**🎯 BÀI TẬP THỰC HÀNH:**
```sql
-- Tạo bảng Orders để luyện tập
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY(1,1),
    UserId INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10,2)
);

INSERT INTO Orders VALUES (1, '2024-01-15', 150.00);
INSERT INTO Orders VALUES (2, '2024-01-16', 200.00);
INSERT INTO Orders VALUES (1, '2024-01-17', 100.00);
INSERT INTO Orders VALUES (3, '2024-01-18', 300.00);

-- BÀI TẬP:
-- 1. Tính tổng doanh thu
-- 2. Tính trung bình giá trị đơn hàng
-- 3. Đếm số đơn hàng của mỗi user
-- 4. Tìm user có tổng giá trị đơn hàng > 200
```

---

## NGÀY 3-4: JOINS - QUAN TRỌNG NHẤT CHO PHỎNG VẤN

### 2.1. INNER JOIN (45 phút)

```sql
-- Chuẩn bị data
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    CustomerName VARCHAR(100)
);

CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    OrderDate DATE,
    Amount DECIMAL(10,2)
);

INSERT INTO Customers VALUES (1, 'John'), (2, 'Alice'), (3, 'Bob');
INSERT INTO Orders VALUES (101, 1, '2024-01-01', 100);
INSERT INTO Orders VALUES (102, 1, '2024-01-02', 150);
INSERT INTO Orders VALUES (103, 2, '2024-01-03', 200);

-- INNER JOIN: Chỉ lấy records có match ở cả 2 bảng
SELECT 
    c.CustomerName,
    o.OrderId,
    o.OrderDate,
    o.Amount
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId;

/*
KẾT QUẢ:
CustomerName | OrderId | OrderDate  | Amount
-------------|---------|------------|--------
John         | 101     | 2024-01-01 | 100
John         | 102     | 2024-01-02 | 150
Alice        | 103     | 2024-01-03 | 200

Bob KHÔNG có trong kết quả vì không có order
*/

-- Query thực tế: Tổng doanh thu theo customer
SELECT 
    c.CustomerName,
    COUNT(o.OrderId) AS TotalOrders,
    SUM(o.Amount) AS TotalSpent
FROM Customers c
INNER JOIN Orders o ON c.CustomerId = o.CustomerId
GROUP BY c.CustomerName;
```

---

### 2.2. LEFT JOIN (30 phút)

```sql
-- LEFT JOIN: Lấy TẤT CẢ records từ bảng bên TRÁI
-- Nếu không match, các cột bên phải = NULL

SELECT 
    c.CustomerId,
    c.CustomerName,
    o.OrderId,
    o.Amount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId;

/*
KẾT QUẢ:
CustomerId | CustomerName | OrderId | Amount
-----------|--------------|---------|--------
1          | John         | 101     | 100
1          | John         | 102     | 150
2          | Alice        | 103     | 200
3          | Bob          | NULL    | NULL   <- Bob vẫn hiện, dù không có order
*/

-- Use case thực tế: Tìm customers CHƯA có order nào
SELECT 
    c.CustomerId,
    c.CustomerName
FROM Customers c
LEFT JOIN Orders o ON c.CustomerId = o.CustomerId
WHERE o.OrderId IS NULL;  -- Orders = NULL nghĩa là chưa đặt hàng

/*
KẾT QUẢ:
CustomerId | CustomerName
-----------|-------------
3          | Bob
*/
```

---

### 2.3. RIGHT JOIN & FULL OUTER JOIN (20 phút)

```sql
-- RIGHT JOIN: Ngược lại với LEFT JOIN
SELECT 
    c.CustomerName,
    o.OrderId,
    o.Amount
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerId = c.CustomerId;
-- Giống LEFT JOIN nhưng đảo ngược vị trí bảng

-- FULL OUTER JOIN: Lấy TẤT CẢ từ cả 2 bảng
SELECT 
    c.CustomerName,
    o.OrderId,
    o.Amount
FROM Customers c
FULL OUTER JOIN Orders o ON c.CustomerId = o.CustomerId;

/*
Lấy:
- Customers có orders
- Customers KHÔNG có orders
- Orders không có customer (nếu có orphan records)
*/
```

---

### 2.4. SELF JOIN (20 phút)

```sql
-- SELF JOIN: Join bảng với chính nó
-- Use case: Employee và Manager (cùng trong 1 bảng)

CREATE TABLE Employees (
    EmployeeId INT PRIMARY KEY,
    EmployeeName VARCHAR(100),
    ManagerId INT
);

INSERT INTO Employees VALUES (1, 'CEO', NULL);
INSERT INTO Employees VALUES (2, 'Manager A', 1);
INSERT INTO Employees VALUES (3, 'Manager B', 1);
INSERT INTO Employees VALUES (4, 'Employee 1', 2);
INSERT INTO Employees VALUES (5, 'Employee 2', 2);

-- Query: Hiển thị employee và manager của họ
SELECT 
    e.EmployeeName AS Employee,
    m.EmployeeName AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;

/*
KẾT QUẢ:
Employee    | Manager
------------|----------
CEO         | NULL
Manager A   | CEO
Manager B   | CEO
Employee 1  | Manager A
Employee 2  | Manager A
*/
```

**🎯 CÂU HỎI PHỎNG VẤN:**
> **"Giải thích sự khác biệt giữa INNER JOIN và LEFT JOIN?"**

**TRẢ LỜI MẪU:**
```
INNER JOIN chỉ lấy records có match ở CẢ 2 bảng.
LEFT JOIN lấy TẤT CẢ records từ bảng bên trái, và match với bảng bên phải.
Nếu không match, các cột bên phải sẽ là NULL.

Ví dụ: Customers LEFT JOIN Orders
→ Sẽ hiển thị cả customers chưa có order nào
```

---

## NGÀY 5: INSERT, UPDATE, DELETE (CRUD Operations)

### 3.1. INSERT - Thêm dữ liệu

```sql
-- Insert 1 record
INSERT INTO Users (UserName, Email, Age, City)
VALUES ('David', 'david@email.com', 28, 'Hanoi');

-- Insert nhiều records cùng lúc
INSERT INTO Users (UserName, Email, Age, City)
VALUES 
    ('Eve', 'eve@email.com', 24, 'HCMC'),
    ('Frank', 'frank@email.com', 32, 'Danang'),
    ('Grace', 'grace@email.com', 27, 'Hanoi');

-- Insert từ SELECT query
INSERT INTO ArchivedUsers (UserName, Email, Age, City)
SELECT UserName, Email, Age, City
FROM Users
WHERE Age > 30;
```

---

### 3.2. UPDATE - Cập nhật dữ liệu

```sql
-- ⚠️ LUÔN LUÔN dùng WHERE khi UPDATE
-- Nếu không có WHERE → Update TẤT CẢ records!

-- Update 1 record
UPDATE Users
SET Age = 26
WHERE UserId = 1;

-- Update nhiều cột
UPDATE Users
SET Age = 26, City = 'HCMC'
WHERE UserId = 1;

-- Update nhiều records
UPDATE Users
SET City = 'Hanoi'
WHERE Age > 30;

-- Update với calculation
UPDATE Orders
SET TotalAmount = TotalAmount * 1.1  -- Tăng 10%
WHERE OrderDate < '2024-01-01';
```

---

### 3.3. DELETE - Xóa dữ liệu

```sql
-- ⚠️ LUÔN LUÔN dùng WHERE khi DELETE
-- Nếu không có WHERE → Xóa TẤT CẢ records!

-- Delete 1 record
DELETE FROM Users WHERE UserId = 1;

-- Delete nhiều records
DELETE FROM Users WHERE Age < 20;

-- Delete tất cả (NGUY HIỂM!)
DELETE FROM Users;  -- Xóa hết nhưng giữ structure

-- TRUNCATE: Xóa nhanh hơn, reset identity
TRUNCATE TABLE Users;
```

**🎯 CÂU HỎI PHỎNG VẤN:**
> **"DELETE vs TRUNCATE khác gì nhau?"**

**TRẢ LỜI:**
```sql
-- DELETE:
-- - Có thể dùng WHERE để xóa từng phần
-- - Chậm hơn với bảng lớn
-- - Có thể ROLLBACK (trong transaction)
-- - Không reset identity counter
DELETE FROM Users WHERE Age < 20;

-- TRUNCATE:
-- - Xóa TẤT CẢ records (không có WHERE)
-- - Nhanh hơn nhiều
-- - Không thể ROLLBACK (trong hầu hết trường hợp)
-- - Reset identity counter về 1
TRUNCATE TABLE Users;
```

---

## NGÀY 6-7: KEYS & CONSTRAINTS - QUAN TRỌNG

### 4.1. PRIMARY KEY

```sql
-- PRIMARY KEY: Unique + NOT NULL, định danh duy nhất
CREATE TABLE Products (
    ProductId INT PRIMARY KEY,
    ProductName VARCHAR(255) NOT NULL
);

-- Hoặc với IDENTITY (auto increment)
CREATE TABLE Products (
    ProductId INT PRIMARY KEY IDENTITY(1,1),  -- Tự động tăng từ 1
    ProductName VARCHAR(255) NOT NULL
);

-- Composite Primary Key (2 cột làm khóa chính)
CREATE TABLE OrderItems (
    OrderId INT,
    ProductId INT,
    Quantity INT,
    PRIMARY KEY (OrderId, ProductId)
);
```

---

### 4.2. FOREIGN KEY

```sql
-- FOREIGN KEY: Tham chiếu đến PRIMARY KEY của bảng khác
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY IDENTITY(1,1),
    CustomerId INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
);

-- Với ON DELETE CASCADE (tự động xóa orders khi xóa customer)
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
        ON DELETE CASCADE  -- Xóa customer → xóa luôn orders
        ON UPDATE CASCADE  -- Update customer ID → update orders
);

-- ON DELETE SET NULL
FOREIGN KEY (CustomerId) REFERENCES Customers(CustomerId)
    ON DELETE SET NULL;  -- Xóa customer → set CustomerId = NULL
```

---

### 4.3. UNIQUE, NOT NULL, DEFAULT, CHECK

```sql
CREATE TABLE Users (
    UserId INT PRIMARY KEY IDENTITY(1,1),
    
    -- UNIQUE: Giá trị không được trùng (nhưng có thể NULL)
    Email VARCHAR(255) UNIQUE,
    
    -- NOT NULL: Bắt buộc phải có giá trị
    UserName VARCHAR(100) NOT NULL,
    
    -- DEFAULT: Giá trị mặc định
    CreatedAt DATETIME DEFAULT GETDATE(),
    IsActive BIT DEFAULT 1,
    
    -- CHECK: Ràng buộc điều kiện
    Age INT CHECK (Age >= 18 AND Age <= 120),
    Status VARCHAR(20) CHECK (Status IN ('Active', 'Inactive', 'Suspended'))
);

-- Test constraints
INSERT INTO Users (Email, UserName, Age, Status)
VALUES ('test@email.com', 'Test User', 25, 'Active');  -- ✅ OK

INSERT INTO Users (Email, UserName, Age, Status)
VALUES ('test@email.com', 'Another User', 25, 'Active');  -- ❌ Error: Email duplicate

INSERT INTO Users (Email, UserName, Age, Status)
VALUES ('test2@email.com', 'Test User', 15, 'Active');  -- ❌ Error: Age < 18
```

**🎯 CÂU HỎI PHỎNG VẤN:**
> **"UNIQUE vs PRIMARY KEY khác gì?"**

**TRẢ LỜI:**
```
- PRIMARY KEY = UNIQUE + NOT NULL
  → Mỗi bảng chỉ có 1 PRIMARY KEY
  → Định danh duy nhất cho record
  → Không thể NULL

- UNIQUE
  → Bảng có thể có nhiều UNIQUE constraints
  → Có thể NULL (và có thể nhiều NULL)
  → Đảm bảo không trùng lặp (trừ NULL)

Ví dụ: Users có thể có:
- PRIMARY KEY: UserId
- UNIQUE: Email (mỗi user 1 email duy nhất)
```

---

## NGÀY 8-9: INDEXES - PERFORMANCE

### 5.1. Tại sao cần Index?

```sql
-- KHÔNG CÓ INDEX: Phải scan toàn bộ bảng
-- Bảng 1 triệu records → Phải đọc hết 1 triệu records

SELECT * FROM Users WHERE Email = 'john@email.com';
-- Execution Plan: Table Scan 🐌 (Chậm)

-- CÓ INDEX: Tìm kiếm nhanh như tra từ điển
CREATE INDEX IX_Users_Email ON Users(Email);

SELECT * FROM Users WHERE Email = 'john@email.com';
-- Execution Plan: Index Seek 🚀 (Nhanh)
```

**Analogy:**
```
Không có Index = Tìm tên trong danh bạ KHÔNG sắp xếp
→ Phải đọc từ trang 1 → trang cuối

Có Index = Tìm tên trong danh bạ ĐÃ sắp xếp A-Z
→ Nhảy thẳng vào đúng chữ cái → Tìm nhanh
```

---

### 5.2. Clustered vs Non-Clustered Index

```sql
-- CLUSTERED INDEX (Chỉ có 1 trên mỗi bảng)
-- Sắp xếp VẬT LÝ dữ liệu trên đĩa
CREATE CLUSTERED INDEX IX_Users_UserId ON Users(UserId);

-- PRIMARY KEY tự động tạo Clustered Index
CREATE TABLE Users (
    UserId INT PRIMARY KEY,  -- Tự động có Clustered Index
    UserName VARCHAR(100)
);

-- NON-CLUSTERED INDEX (Có thể có nhiều)
-- Tạo cấu trúc riêng trỏ đến data
CREATE NONCLUSTERED INDEX IX_Users_Email ON Users(Email);
CREATE NONCLUSTERED INDEX IX_Users_City ON Users(City);

-- Composite Index (nhiều cột)
CREATE NONCLUSTERED INDEX IX_Users_City_Age ON Users(City, Age);

-- Query được tối ưu với composite index
SELECT * FROM Users WHERE City = 'Hanoi' AND Age = 25;
```

**🎯 CÂU HỎI PHỎNG VẤN:**
> **"Khi nào nên tạo Index? Khi nào KHÔNG nên?"**

**TRẢ LỜI:**
```
✅ NÊN TẠO INDEX KHI:
- Cột thường xuyên xuất hiện trong WHERE
- Cột trong JOIN conditions
- Cột trong ORDER BY
- Bảng lớn (nhiều records)
- Read nhiều hơn Write

❌ KHÔNG NÊN TẠO INDEX KHI:
- Bảng nhỏ (< 1000 records)
- Cột ít unique values (VD: Gender - chỉ có M/F)
- Write nhiều hơn Read (mỗi INSERT/UPDATE/DELETE phải update index)
- Cột không bao giờ xuất hiện trong WHERE/JOIN

Trade-off:
+ Index tăng tốc SELECT
- Index làm chậm INSERT/UPDATE/DELETE
- Index tốn thêm storage
```

---

## NGÀY 10: TRANSACTIONS - ACID

### 6.1. Transaction là gì?

```sql
-- Transaction = Nhóm các operations thành 1 đơn vị
-- Hoặc TẤT CẢ thành công, hoặc TẤT CẢ fail (không có "một nửa")

-- VÍ DỤ: Chuyển tiền giữa 2 tài khoản
BEGIN TRANSACTION;

    -- Bước 1: Trừ tiền tài khoản A
    UPDATE Accounts 
    SET Balance = Balance - 100 
    WHERE AccountId = 1;
    
    -- Bước 2: Cộng tiền tài khoản B
    UPDATE Accounts 
    SET Balance = Balance + 100 
    WHERE AccountId = 2;
    
    -- Nếu cả 2 bước OK → COMMIT
    COMMIT TRANSACTION;

-- Nếu có lỗi → ROLLBACK (hoàn tác tất cả)
```

---

### 6.2. Transaction với Error Handling

```sql
BEGIN TRY
    BEGIN TRANSACTION;
    
        -- Kiểm tra số dư
        DECLARE @Balance DECIMAL(10,2);
        SELECT @Balance = Balance FROM Accounts WHERE AccountId = 1;
        
        IF @Balance < 100
        BEGIN
            -- Không đủ tiền → ROLLBACK
            THROW 50001, 'Insufficient funds', 1;
        END
        
        -- Trừ tiền A
        UPDATE Accounts SET Balance = Balance - 100 WHERE AccountId = 1;
        
        -- Cộng tiền B
        UPDATE Accounts SET Balance = Balance + 100 WHERE AccountId = 2;
        
    COMMIT TRANSACTION;
    PRINT 'Transfer successful';
    
END TRY
BEGIN CATCH
    -- Có lỗi → ROLLBACK
    ROLLBACK TRANSACTION;
    PRINT 'Error: ' + ERROR_MESSAGE();
END CATCH;
```

**🎯 CÂU HỎI PHỎNG VẤN:**
> **"ACID là gì?"**

**TRẢ LỜI:**
```
A - Atomicity (Tính nguyên tử)
→ Tất cả hoặc không có gì (all or nothing)

C - Consistency (Tính nhất quán)
→ Database luôn ở trạng thái hợp lệ
→ Constraints luôn được đảm bảo

I - Isolation (Tính độc lập)
→ Các transactions không ảnh hưởng lẫn nhau
→ Transaction A không thấy dữ liệu chưa commit của Transaction B

D - Durability (Tính bền vững)
→ Sau khi COMMIT, dữ liệu được lưu vĩnh viễn
→ Ngay cả khi server crash

Ví dụ: Chuyển tiền
- A: Hoặc cả 2 tài khoản đều update, hoặc không tài khoản nào update
- C: Tổng tiền trước = Tổng tiền sau
- I: 2 giao dịch chuyển tiền cùng lúc không làm loạn số dư
- D: Sau khi chuyển xong, dữ liệu không mất dù server tắt
```

---

## NGÀY 11-12: SUBQUERIES & ADVANCED

### 7.1. Subquery trong WHERE

```sql
-- Tìm users có order với giá trị > 200
SELECT * FROM Users
WHERE UserId IN (
    SELECT CustomerId FROM Orders WHERE TotalAmount > 200
);

-- Tìm products có giá cao hơn giá trung bình
SELECT * FROM Products
WHERE Price > (SELECT AVG(Price) FROM Products);

-- EXISTS: Kiểm tra sự tồn tại
SELECT * FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o 
    WHERE o.CustomerId = c.CustomerId
);
-- Faster than IN cho large datasets

-- NOT EXISTS: Ngược lại
SELECT * FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o 
    WHERE o.CustomerId = c.CustomerId
);
-- Customers chưa có order nào
```

---

### 7.2. Subquery trong SELECT

```sql
-- Hiển thị customer và tổng số orders
SELECT 
    c.CustomerName,
    (SELECT COUNT(*) FROM Orders o WHERE o.CustomerId = c.CustomerId) AS TotalOrders
FROM Customers c;

-- Hiển thị product và giá so với giá trung bình
SELECT 
    ProductName,
    Price,
    (SELECT AVG(Price) FROM Products) AS AvgPrice,
    Price - (SELECT AVG(Price) FROM Products) AS PriceDifference
FROM Products;
```

---

### 7.3. Common Table Expressions (CTE)

```sql
-- CTE: Như một "temporary result set"
-- Dễ đọc hơn subquery

-- Tìm top 3 customers theo tổng chi tiêu
WITH CustomerSpending AS (
    SELECT 
        CustomerId,
        SUM(TotalAmount) AS TotalSpent
    FROM Orders
    GROUP BY CustomerId
)
SELECT TOP 3
    c.CustomerName,
    cs.TotalSpent
FROM CustomerSpending cs
INNER JOIN Customers c ON cs.CustomerId = c.CustomerId
ORDER BY cs.TotalSpent DESC;

-- Recursive CTE: Employee hierarchy
WITH EmployeeHierarchy AS (
    -- Anchor: Top level
    SELECT EmployeeId, ManagerId, EmployeeName, 0 AS Level
    FROM Employees
    WHERE ManagerId IS NULL
    
    UNION ALL
    
    -- Recursive: Next levels
    SELECT e.EmployeeId, e.ManagerId, e.EmployeeName, eh.Level + 1
    FROM Employees e
    INNER JOIN EmployeeHierarchy eh ON e.ManagerId = eh.EmployeeId
)
SELECT * FROM EmployeeHierarchy;
```

---

## NGÀY 13-14: SQL SERVER SPECIFICS

### 8.1. IDENTITY (Auto Increment)

```sql
-- IDENTITY(start, increment)
CREATE TABLE Products (
    ProductId INT IDENTITY(1,1) PRIMARY KEY,  -- 1, 2, 3, 4...
    ProductName VARCHAR(255)
);

-- Lấy ID vừa insert
INSERT INTO Products (ProductName) VALUES ('Product A');
SELECT SCOPE_IDENTITY();  -- Returns last inserted ID

-- Reset IDENTITY
DBCC CHECKIDENT ('Products', RESEED, 0);  -- Reset về 0
```

---

### 8.2. Variables & Control Flow

```sql
-- Declare variables
DECLARE @UserCount INT;
DECLARE @Message VARCHAR(100);

-- Set values
SET @UserCount = (SELECT COUNT(*) FROM Users);
SET @Message = 'Total users: ' + CAST(@UserCount AS VARCHAR);

-- IF...ELSE
IF @UserCount > 100
    PRINT 'We have many users!';
ELSE
    PRINT 'We need more users.';

-- WHILE loop
DECLARE @Counter INT = 1;
WHILE @Counter <= 10
BEGIN
    PRINT 'Counter: ' + CAST(@Counter AS VARCHAR);
    SET @Counter = @Counter + 1;
END
```

---

### 8.3. Stored Procedures (Cơ bản)

```sql
-- Tạo stored procedure đơn giản
CREATE PROCEDURE sp_GetUserById
    @UserId INT
AS
BEGIN
    SELECT * FROM Users WHERE UserId = @UserId;
END

-- Gọi stored procedure
EXEC sp_GetUserById @UserId = 1;

-- Stored procedure với OUTPUT parameter
CREATE PROCEDURE sp_GetUserCount
    @City VARCHAR(100),
    @UserCount INT OUTPUT
AS
BEGIN
    SELECT @UserCount = COUNT(*) FROM Users WHERE City = @City;
END

-- Gọi với OUTPUT
DECLARE @Count INT;
EXEC sp_GetUserCount @City = 'Hanoi', @UserCount = @Count OUTPUT;
PRINT 'Users in Hanoi: ' + CAST(@Count AS VARCHAR);
```

---

### 8.4. Views

```sql
-- View: "Virtual table" lưu query
CREATE VIEW vw_ActiveUsers AS
SELECT UserId, UserName, Email, City
FROM Users
WHERE IsActive = 1;

-- Sử dụng view như table bình thường
SELECT * FROM vw_ActiveUsers;
SELECT * FROM vw_ActiveUsers WHERE City = 'Hanoi';

-- Update view
ALTER VIEW vw_ActiveUsers AS
SELECT UserId, UserName, Email, City, Age
FROM Users
WHERE IsActive = 1 AND Age >= 18;

-- Drop view
DROP VIEW vw_ActiveUsers;
```

---

## CHIẾN LƯỢC ÔN TẬP CẤP TỐC

### 📅 Nếu còn 7 NGÀY:

**Ngày 1-2: FOUNDATION**
- [ ] SELECT, WHERE, ORDER BY (1 giờ)
- [ ] Aggregate Functions: COUNT, SUM, AVG (1 giờ)
- [ ] GROUP BY, HAVING (1 giờ)
- [ ] Làm 20 bài LeetCode Easy SQL

**Ngày 3-4: JOINS (QUAN TRỌNG NHẤT)**
- [ ] INNER JOIN (2 giờ)
- [ ] LEFT JOIN (1 giờ)
- [ ] SELF JOIN (1 giờ)
- [ ] Làm 15 bài LeetCode Medium về JOINs

**Ngày 5: CRUD & KEYS**
- [ ] INSERT, UPDATE, DELETE (1 giờ)
- [ ] PRIMARY KEY, FOREIGN KEY (1 giờ)
- [ ] Constraints: UNIQUE, NOT NULL, CHECK (30 phút)

**Ngày 6: ADVANCED**
- [ ] Subqueries (1 giờ)
- [ ] CTE (1 giờ)
- [ ] Indexes (30 phút - lý thuyết)

**Ngày 7: MOCK INTERVIEW**
- [ ] Làm 10 câu hỏi phỏng vấn thực tế
- [ ] Review lại tất cả concepts
- [ ] Prepare câu trả lời cho "Tại sao dùng index", "ACID", "JOIN types"

---

### 📅 Nếu còn 3 NGÀY (Cấp cứu):

**Ngày 1: CORE CONCEPTS (8 giờ)**
- ☑️ SELECT, WHERE, ORDER BY (30 phút)
- ☑️ Aggregate + GROUP BY (1 giờ)
- ☑️ INNER JOIN + LEFT JOIN (2 giờ) ← QUAN TRỌNG NHẤT
- ☑️ PRIMARY/FOREIGN KEY (30 phút)
- ☑️ Làm 30 bài LeetCode Easy+Medium (4 giờ)

**Ngày 2: PRACTICE (8 giờ)**
- ☑️ Làm 50 câu SQL trên LeetCode/HackerRank
- ☑️ Focus: JOINs, GROUP BY, Subqueries

**Ngày 3: MOCK + THEORY (8 giờ)**
- ☑️ Làm 20 câu phỏng vấn thực tế (4 giờ)
- ☑️ Học thuộc lòng (4 giờ):
  - ACID là gì
  - Clustered vs Non-clustered index
  - INNER vs LEFT JOIN
  - PRIMARY KEY vs UNIQUE
  - Transaction basics

---

### 📅 Nếu còn 1 NGÀY (Khẩn cấp):

**Sáng (4 giờ): CONCEPTS QUAN TRỌNG NHẤT**
1. SELECT + WHERE + JOINs (2 giờ)
2. Aggregate Functions + GROUP BY (1 giờ)
3. PRIMARY KEY, FOREIGN KEY, Index (1 giờ)

**Chiều (4 giờ): PRACTICE**
1. Làm 30 câu SQL LeetCode Top Interview Questions
2. Focus 100% vào JOINs

**Tối (2 giờ): MEMORIZE**
Học thuộc:
- ACID
- Index là gì, khi nào dùng
- INNER JOIN vs LEFT JOIN (giải thích được)
- PRIMARY KEY vs FOREIGN KEY

---

## 10 CÂU HỎI PHỎNG VẤN PHẢI BIẾT

### Câu 1: "Giải thích INNER JOIN vs LEFT JOIN với ví dụ"
```sql
-- Sample data
Customers: 1-John, 2-Alice, 3-Bob
Orders: 101(Customer 1), 102(Customer 1), 103(Customer 2)

-- INNER JOIN: Chỉ customers CÓ orders
SELECT c.CustomerName, o.OrderId
FROM Customers c INNER JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Result: John (2 orders), Alice (1 order)
-- Bob KHÔNG có vì không có order

-- LEFT JOIN: TẤT CẢ customers
SELECT c.CustomerName, o.OrderId
FROM Customers c LEFT JOIN Orders o ON c.CustomerId = o.CustomerId;
-- Result: John (2 orders), Alice (1 order), Bob (NULL)
```

### Câu 2: "PRIMARY KEY vs FOREIGN KEY?"
```
PRIMARY KEY:
- Định danh duy nhất cho record
- UNIQUE + NOT NULL
- Mỗi bảng chỉ 1 PRIMARY KEY
VD: UserId trong bảng Users

FOREIGN KEY:
- Tham chiếu đến PRIMARY KEY của bảng khác
- Đảm bảo referential integrity
- Có thể NULL
VD: CustomerId trong bảng Orders → tham chiếu Users(UserId)
```

### Câu 3: "DELETE vs TRUNCATE?"
```
DELETE:
- Xóa từng row (có thể dùng WHERE)
- Có thể ROLLBACK
- Không reset IDENTITY
- Chậm hơn

TRUNCATE:
- Xóa tất cả rows (không có WHERE)
- Không thể ROLLBACK
- Reset IDENTITY về 1
- Nhanh hơn
```

### Câu 4: "Index là gì? Khi nào dùng?"
```
Index = Cấu trúc data giúp tìm kiếm nhanh hơn
Giống như mục lục trong sách

Khi nào dùng:
✅ Cột trong WHERE, JOIN, ORDER BY
✅ Bảng lớn (> 10k records)
✅ Read nhiều hơn Write

Khi nào KHÔNG dùng:
❌ Bảng nhỏ
❌ Cột ít unique values
❌ Write nhiều hơn Read
```

### Câu 5: "ACID là gì?"
```
A - Atomicity: All or nothing (tất cả hoặc không)
C - Consistency: Database luôn hợp lệ
I - Isolation: Transactions không ảnh hưởng nhau
D - Durability: Dữ liệu được lưu vĩnh viễn sau COMMIT

VD: Chuyển tiền - hoặc cả 2 tài khoản đều update, hoặc không ai update
```

### Câu 6: "GROUP BY vs HAVING?"
```sql
SELECT City, COUNT(*) AS UserCount
FROM Users
WHERE Age > 20        -- WHERE: Lọc TRƯỚC khi group
GROUP BY City
HAVING COUNT(*) > 10; -- HAVING: Lọc SAU khi group

-- WHERE filter rows, HAVING filter groups
```

### Câu 7: "Subquery vs JOIN - Khi nào dùng?"
```sql
-- Subquery: Đơn giản, dễ đọc
SELECT * FROM Users
WHERE UserId IN (SELECT CustomerId FROM Orders);

-- JOIN: Nhanh hơn, linh hoạt hơn
SELECT DISTINCT u.*
FROM Users u
INNER JOIN Orders o ON u.UserId = o.CustomerId;

→ Nên dùng JOIN khi có thể (performance tốt hơn)
```

### Câu 8: "N+1 Query Problem là gì?"
```
Problem: Query 1 lần để lấy N records, 
rồi query N lần nữa để lấy related data → Tổng N+1 queries

Solution: Dùng JOIN để lấy tất cả trong 1 query

-- BAD: 1 + 100 queries
SELECT * FROM Users; -- 100 users
For each user: SELECT * FROM Orders WHERE CustomerId = ...

-- GOOD: 1 query
SELECT u.*, o.* FROM Users u LEFT JOIN Orders o ON u.UserId = o.CustomerId;
```

### Câu 9: "Normalization là gì?"
```
= Tổ chức data để giảm redundancy (dư thừa)

1NF: Mỗi cell chỉ 1 giá trị (không có arrays)
2NF: Không có partial dependency
3NF: Không có transitive dependency

VD: Thay vì lưu CustomerName trong Orders
→ Lưu CustomerId, JOIN với Customers để lấy name
```

### Câu 10: "Optimize slow query như thế nào?"
```
1. Check Execution Plan (Ctrl + M trong SSMS)
2. Tìm Table Scan → Thêm Index
3. Tránh SELECT * → Chỉ lấy cột cần thiết
4. Tránh functions trong WHERE: WHERE YEAR(OrderDate) = 2024
   → Dùng: WHERE OrderDate >= '2024-01-01' AND OrderDate < '2025-01-01'
5. Dùng EXISTS thay IN với subquery lớn
6. Update Statistics nếu cần
```

---

## TÀI NGUYÊN HỌC TẬP

### 🎯 Practice Platforms (ƯU TIÊN):
1. **LeetCode Database** - https://leetcode.com/problemset/database/
   - Top Interview Questions: BẮT BUỘC phải làm
   
2. **HackerRank SQL** - https://www.hackerrank.com/domains/sql
   - Easy → Medium → Hard

3. **SQLZoo** - https://sqlzoo.net/
   - Interactive tutorials

### 📚 Tài liệu tham khảo nhanh:
- W3Schools SQL: https://www.w3schools.com/sql/
- SQL Server Documentation: https://docs.microsoft.com/sql/

### 🛠️ Tools cần cài:
- **SQL Server Express** (Free): https://www.microsoft.com/sql-server/sql-server-downloads
- **SQL Server Management Studio (SSMS)**: https://aka.ms/ssmsfullsetup
- **Azure Data Studio** (Alternative): https://docs.microsoft.com/sql/azure-data-studio/

---

## CHECKLIST TRƯỚC PHỎNG VẤN

### ✅ Kiến thức PHẢI BIẾT:
- [ ] SELECT, WHERE, ORDER BY
- [ ] Aggregate Functions (COUNT, SUM, AVG, MAX, MIN)
- [ ] GROUP BY, HAVING
- [ ] INNER JOIN, LEFT JOIN
- [ ] PRIMARY KEY, FOREIGN KEY
- [ ] INSERT, UPDATE, DELETE
- [ ] Index cơ bản
- [ ] Transaction cơ bản

### ✅ Có thể giải thích:
- [ ] INNER JOIN vs LEFT JOIN
- [ ] PRIMARY KEY vs UNIQUE
- [ ] DELETE vs TRUNCATE
- [ ] Index là gì, khi nào dùng
- [ ] ACID là gì
- [ ] WHERE vs HAVING
- [ ] Subquery vs JOIN

### ✅ Đã luyện tập:
- [ ] 50+ câu SQL trên LeetCode/HackerRank
- [ ] Có thể viết JOIN query phức tạp
- [ ] Có thể viết GROUP BY với aggregate functions
- [ ] Đã làm ít nhất 10 câu mock interview

---

## TIPS PHỎNG VẤN

### 1. Khi được hỏi viết query:
```
✅ Đọc kỹ yêu cầu
✅ Vẽ diagram nếu có nhiều bảng
✅ Viết query từng bước
✅ Test với sample data (nếu có whiteboard)
✅ Giải thích logic trong đầu
```

### 2. Nếu không biết:
```
✅ Thừa nhận thẳng thắn: "Em chưa có kinh nghiệm với feature này"
✅ Nói về approach: "Em nghĩ có thể làm theo hướng..."
✅ Hỏi lại: "Anh có thể cho em hint được không?"
❌ ĐỪNG bịa đặt hoặc nói dối
```

### 3. Câu hỏi nên hỏi lại interviewer:
```
- Database schema như thế nào?
- Bảng có bao nhiêu records?
- Có index nào đã được tạo chưa?
- Performance requirement là gì?
```

---

## TÓM TẮT - SHORTEST PATH TO PASS INTERVIEW

### TOP 5 CONCEPTS PHẢI BIẾT:
1. **JOINs** (INNER, LEFT) - 40% câu hỏi
2. **GROUP BY + Aggregates** - 30%
3. **Subqueries** - 15%
4. **Keys & Constraints** - 10%
5. **Indexes** - 5%

### 3 ĐIỀU QUAN TRỌNG NHẤT:
1. **Practice JOINs** - Làm cho đến khi thành reflex
2. **Understand Execution Plans** - Biết query nào chậm, tại sao chậm
3. **Real-world scenarios** - Không chỉ syntax, phải hiểu khi nào dùng

### GOLDEN RULE:
```
Interviewer không mong bạn biết TẤT CẢ
Nhưng bạn PHẢI:
1. Biết basics vững vàng (SELECT, JOIN, GROUP BY)
2. Có thể giải thích logic
3. Sẵn sàng học hỏi

→ "Em chưa dùng feature này trong project trước, 
    nhưng em hiểu concept là... và em sẵn sàng học thêm"
```

---

**GOOD LUCK! Bạn làm được! 💪**

Remember: SQL không khó, chỉ cần practice! Làm 50-100 bài là đã confident rồi!
