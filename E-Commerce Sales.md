# SQL-Project
## Cleaning Data
Use [E-Commerce Sales Dataset]
update [Amazon Sale Report]
set 
	Amount = 0
where 
	Amount = ' ';
update [Amazon Sale Report]
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
DELETE FROM CTE WHERE rn > 1;
Delete From [Amazon Sale Report]
Where
	[Order ID] is NULL
	Or Date is NULL
	Or Status is NULL;
Update [Amazon Sale Report]
Set 
	shipcity = lower(shipcity),
	shipstate = lower(shipstate);
Use [E-Commerce Sales Dataset]
ALTER TABLE [Amazon Sale Report]
ALTER COLUMN Amount DECIMAL(10, 2);
ALTER TABLE [Amazon Sale Report]
ALTER COLUMN Qty DECIMAL(10, 2);

Delete from [May-2022]
Where
	[Ajio MRP]  = 'Nill' or
	[Amazon MRP] = 'Nill';
With CTE2 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY Sku, [Style Id], Catalog, Category, Weight, TP ORDER BY Sku) AS rn2
    FROM [May-2022]
)
DELETE FROM CTE2 WHERE rn2 > 1;


Delete from [P_L March 2021]
Where
	[Ajio MRP]  = 'Nill' or
	[Amazon MRP] = 'Nill';
With CTE3 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY Sku, [Style Id], Catalog, Category, Weight, [TP 1], [TP 2] ORDER BY Sku) AS rn3
    FROM [P_L March 2021]
)
DELETE FROM CTE3 WHERE rn3 > 1;

Delete from [Sale Report]
Where [SKU Code] = '#REF!'
	or [SKU Code] IS NULL 
	or [SKU Code] = ' ';
With CTE4 As (
	SELECT *, ROW_NUMBER() OVER (PARTITION BY [SKU Code], [Design No], Stock, Category, Size, Color ORDER BY [Design No]) AS rn4
    FROM [Sale Report]
)
DELETE FROM CTE4 WHERE rn4 > 1;

Delete from [International sale Report]
Where SKU = ' '
	or SKU = 'SHIPPING'
	or SKU = 'TAGS'
	or SKU = 'TAG PRINTING'
	or SKU = 'TAGS(LABOUR)'
	or SKU = 'SHIPPING CHARGES';
With CTE5 as (
	Select *, Row_number() over (Partition by SKU, Style, CUSTOMER, Months, DATE, Size, PCS, RATE, [GROSS AMT] Order by DATE) as rn5
	from [International sale Report]
)
Delete from CTE5 Where rn5 >1;
DELETE FROM [dbo].[International sale Report]
WHERE TRY_CONVERT(DATE, [DATE]) IS NULL;
ALTER TABLE [International sale Report]
	ALTER COLUMN [GROSS AMT] DECIMAL(10, 2);
ALTER TABLE [International sale Report]
	ALTER COLUMN PCS DECIMAL(10, 2);


## Answer Business Questions
-- Q1. Viết lệnh truy vấn doanh thu Category giao thành công và có doanh thu của Amazon--
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

-- Q2. Viết lệnh truy vấn thống kê số lượng sản phẩm bán chạy theo kích cỡ, màu sắc--
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

-- Q3. Viết lệnh truy vấn 10 thành phố có nhiều đơn thành công nhất. Sắp xếp theo thứ tự từ cao xuống thấp--
Select distinct top 10
	shipcity,
	COUNT ([Order ID]) as tong_so_don
From [Amazon Sale Report]
Where [Courier Status] = 'Shipped'
Group by shipcity
Order by dem DESC;

--Q4. Viết lệnh truy vấn 10 thành phố có doanh số bán hàng lớn nhất. Sắp xếp doanh số theo thứ tự giảm dần
Select distinct top 10
	shipcity,
	Sum (Amount) as total_sale
From [Amazon Sale Report]
Where 
	[Courier Status] = 'Shipped'
Group by shipcity
Order by total_sale DESC;

--Q5. Viết lệnh truy vấn tổng doanh thu doanh thu và tổng số lượng khách hàng theo từng tháng
Select 
	MONTH(a.Date) as order_month,
	Sum(a.Amount) as total_sales,
	COUNT(b.CUSTOMER) as total_customer
From [Amazon Sale Report] as a
Left Join [International sale Report] as b on MONTH(a.Date) = MONTH(b.DATE)
Where a.[Courier Status] = 'Shipped'
Group by MONTH(a.Date)
Order by MONTH(a.Date)

--Q6. Viết lệnh truy vấn tỷ trọng của từng danh mục trong tổng doanh thu
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

--Q7. Viết truy vấn tính tổng chi tiêu, số lượng đơn hàng, và sản phẩm độc nhất theo thành phố
SELECT 
    shipcity,
    COUNT(DISTINCT SKU) AS UniqueProducts,
    SUM(Amount) AS TotalSpent,
    COUNT(*) AS OrderCount
FROM [Amazon Sale Report]
WHERE [Courier Status] = 'Shipped'
GROUP BY shipcity
ORDER BY TotalSpent DESC;

--Q8.Viết truy vấn tính tổng số lượng bán, doanh thu, và giá trung bình theo SKU
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
