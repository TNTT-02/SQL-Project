# SQL-Project
## Cleaning Data
Use [E-Commerce Sales Dataset]
-- Thao tác với bảng Amazon Sale Report--
update [Amazon Sale Report] --> Điền các giá trị trống ở cột Amount bằng giá trị 0
set 
	Amount = 0
where 
	Amount = ' ';
update [Amazon Sale Report]--> Điền các giá trị trống ở cột Courier Status bằng giá trị Unshipped
set 
	[Courier Status] = 'Unshipped'
Where 
	[Courier Status] = ' ' and 
	Status = 'Cancelled';
update [Amazon Sale Report]
Set 
	currency = 'INR'
Where
	currency = ' ';
WITH CTE AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY [Order ID], Status, Date, SKU, Amount ORDER BY Date) AS rn
    FROM [Amazon Sale Report]
)
DELETE FROM CTE WHERE rn > 1;--> Xóa dòng dữ liệu bị trùng lặp
Delete From [Amazon Sale Report] --> Xóa dòng dữ liệu có giá trị NUll
Where
	[Order ID] is NULL
	Or Date is NULL
	Or Status is NULL;
Update [Amazon Sale Report] --> Chuẩn hóa cột shipcity và shipstate
Set 
	shipcity = lower(shipcity),
	shipstate = lower(shipstate);
Use [E-Commerce Sales Dataset]
ALTER TABLE [Amazon Sale Report]
ALTER COLUMN Amount DECIMAL(10, 2); --> Đổi kiểu dữ liệu của cột Amount
ALTER TABLE [Amazon Sale Report]
ALTER COLUMN Qty DECIMAL(10, 2); --> Đổi kiểu dữ liệu của cột Qty
--Thao tác với bảng May-2022--
Delete from [May-2022]
Where
	[Ajio MRP]  = 'Nill' or
	[Amazon MRP] = 'Nill';
With CTE2 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY Sku, [Style Id], Catalog, Category, Weight, TP ORDER BY Sku) AS rn2
    FROM [May-2022]
)
DELETE FROM CTE2 WHERE rn2 > 1; --> Xóa dòng dữ liệu bị trùng lặp
--Thao tác với bảng P_L March 2021--
Delete from [P_L March 2021]
Where
	[Ajio MRP]  = 'Nill' or
	[Amazon MRP] = 'Nill';
With CTE3 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY Sku, [Style Id], Catalog, Category, Weight, [TP 1], [TP 2] ORDER BY Sku) AS rn3
    FROM [P_L March 2021]
)
DELETE FROM CTE3 WHERE rn3 > 1;--> Xóa dòng dữ liệu bị trùng lặp
--Thao tác với bảng Sale Report--
Delete from [Sale Report]
Where [SKU Code] = '#REF!'
	or [SKU Code] IS NULL 
	or [SKU Code] = ' ';--> Xóa dòng dữ liệu bị lỗi hoặc NULL
With CTE4 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY [SKU Code], [Design No], Stock, Category, Size, Color ORDER BY [Design No]) AS rn4
    FROM [Sale Report]
)
DELETE FROM CTE4 WHERE rn4 > 1;--> Xóa dòng dữ liệu bị trùng lặp
-- Thao tác với bảng Interntional sale Report --
Delete from [International sale Report]
Where SKU = ' '
	or SKU = 'SHIPPING'
	or SKU = 'TAGS'
	or SKU = 'TAG PRINTING'
	or SKU = 'TAGS(LABOUR)'
	or SKU = 'SHIPPING CHARGES';--> Xóa dòng dữ liệu không đúng định dạng
With CTE5 as (
	Select *, Row_number() over (Partition by SKU, Style, CUSTOMER, Months, DATE, Size, PCS, RATE, [GROSS AMT] Order by DATE) as rn5
	from [International sale Report]
)
Delete from CTE5 Where rn5 >1;--> Xóa dòng dữ liệu bị trùng lặp
DELETE FROM [dbo].[International sale Report]
WHERE TRY_CONVERT(DATE, [DATE]) IS NULL;
ALTER TABLE [International sale Report]
	ALTER COLUMN [GROSS AMT] DECIMAL(10, 2);--> Đổi kiểu dữ liệu cột GROSS AMT
ALTER TABLE [International sale Report]
	ALTER COLUMN PCS DECIMAL(10, 2);--> Đổi kiểu dữ liệu cột PCS
## Database Exploration
Purpose:
    - To explore the structure of the database, including the list of tables and their schemas.
    - To inspect the columns and metadata for specific tables.
    - To explore the structure of dimension tables.
    - To understand the range of historical data.
-- Retrieve a list of all tables in the database
Use [E-Commerce Sales Dataset]
Select *
From INFORMATION_SCHEMA.TABLES;
-- Retrieve all columns for a specific table
Use [E-Commerce Sales Dataset]
Select *
From [Amazon Sale Report]
-- Get a list of unique cities where orders go
Use [E-Commerce Sales Dataset]
Select distinct
	shipcity
From [Amazon Sale Report]
Order by shipcity;
-- Get a list of months in the data
Use [E-Commerce Sales Dataset]
Select distinct
	MONTH(DATE) as thang
From [Amazon Sale Report]
Order by MONTH(DATE);
## Answer Business Questions
-- Q1. Write a query for Category revenue that is successfully delivered and has Amazon revenue--
Use [E-Commerce Sales Dataset]
Select
	[Order ID],
	Date,
	Category,
	Amount
From [Amazon Sale Report]
Where 
	Status in ('Shipped', 'Shipped - Delivered to Buyer', 'Shipped - Returned to Seller')
	And Amount >0
Group by Category, Amount, [Order ID], Date
Order by Category ASC;

-- Q2. Write a query to count the number of best-selling products by size and color--
With cte1 as (
	Select 
		a.SKU, 
		a.Style, 
		c.Color,
		a.Size,
		b.Category,
		a.PCS,
		a.[GROSS AMT],
		Count(*) over (Partition by a.Style, c.Color,b.Category Order by a.[GROSS AMT]) as cp1
From [International sale Report] as a
Inner join [Amazon Sale Report] as b on a.SKU = b.SKU
Inner join [Sale Report] as c on a.SKU = c.[SKU Code]
Where b.[Courier Status] = 'Shipped'
)
Select distinct
	SKU,
	Style,
	Size,
	Color,
	Category,
	Count(PCS) as tong_so_ban,
	sum([GROSS AMT]) as total_sale,
	COUNT (SKU) as tong_so_don
From cte1
Group by SKU, Style, Size, Color, Category
Order by tong_so_don DESC;

-- Q3. Write a query to find the 10 cities with the most successful orders. Sort from highest to lowest--
Select distinct top 10
	shipcity,
	COUNT ([Order ID]) as tong_so_don
From [Amazon Sale Report]
Where [Courier Status] = 'Shipped'
Group by shipcity
Order by tong_so_don DESC;

--Q4. Viết lệnh truy vấn 10 thành phố có doanh số bán hàng lớn nhất. Sắp xếp doanh số theo thứ tự giảm dần --
Select distinct top 10
	shipcity,
	Sum (Amount) as total_sale
From [Amazon Sale Report]
Where 
	[Courier Status] = 'Shipped'
Group by shipcity
Order by total_sale DESC;

--Q5. Write a query for total revenue and total number of customers by month --
Select 
	MONTH(a.Date) as order_month,
	Sum(a.Amount) as total_sales,
	COUNT(b.CUSTOMER) as total_customer
From [Amazon Sale Report] as a
Left Join [International sale Report] as b on MONTH(a.Date) = MONTH(b.DATE)
Where a.[Courier Status] = 'Shipped'
Group by MONTH(a.Date)
Order by MONTH(a.Date)

--Q6. Write a query to find the proportion of each category in total revenue --
WITH cte2 AS (
    SELECT 
        Category,
        SUM(Amount) AS CategoryTotal,
        (SELECT SUM(Amount) FROM [Amazon Sale Report] WHERE [Courier Status] = 'Shipped') AS GrandTotal
    FROM [Amazon Sale Report]
    WHERE [Courier Status] = 'Shipped'
    GROUP BY Category
)
SELECT 
    Category,
    CategoryTotal,
    GrandTotal,
    (CategoryTotal * 100.0 / GrandTotal) AS Percentage
FROM cte2;

--Q7. Write a query to calculate total spend, number of orders, and unique products by city --
SELECT 
    shipcity,
    COUNT(DISTINCT SKU) AS UniqueProducts,
    SUM(Amount) AS TotalSpent,
    COUNT(*) AS OrderCount
FROM [Amazon Sale Report]
WHERE [Courier Status] = 'Shipped'
GROUP BY shipcity
ORDER BY TotalSpent DESC;

--Q8.Write a query to calculate the total sales quantity, revenue, and average price by SKU --
SELECT 
    SKU,
    Category,
    SUM(Qty) AS TotalSold,
    SUM(Amount) AS TotalRevenue,
    AVG(Amount / Qty) AS AvgPricePerUnit
FROM [Amazon Sale Report]
WHERE [Courier Status] = 'Shipped'
GROUP BY SKU, Category
ORDER BY TotalSold DESC;
