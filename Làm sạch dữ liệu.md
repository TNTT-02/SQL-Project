# SQL-Project
Làm sạch bằng SQL
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
ALTER TABLE [Amazon Sale Report]
ALTER COLUMN Amount DECIMAL(10, 2);

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
