Data Analysis Project - Sales Management
1 - Introduction & Setup
Adventure Works is a fictional company, a large multinational manufacturing company that produces and distributes bicycles, bike parts, and accessories. It operates in the North American, European, and Asian markets and has 500 employees. Adventure Works has various sales groups.

The link to download Adventure Works' Data Warehouse is provided below.

Adventure Works Data Warehouse

For the ETL process, SQL Server was used, and Power BI was used for data visualization. The database was last updated in 2014, and a SQL script from my GitHub was used to update it.

2 - Business Requirements and Customer Needs
Business requirements and customer needs were established by the Sales Manager. Based on the employee's requirements, the table below was generated, containing acceptance criteria and details for dashboard production.

#	Position	Demand	Objective	Acceptance Criteria
1	Sales Manager	Produce a dashboard	Improve tracking of top customers and products	A report that updates daily
2	Sales Representative	Detailed internet sales	Monitor top-buying customers and identify opportunities	A dashboard that allows filtering by customer
3	Sales Representative	Detailed internet sales	Monitor high-performing products in sales	A dashboard that allows filtering by product (category and subcategory)
4	Sales Manager	Produce a dashboard	Monitor internet sales and budget over time	A chart and a KPI comparing sales and budget over time
3 - Data Cleaning and Transformation (SQL)
In this stage, SQL Server was used to access the Data Warehouse and obtain the necessary data for dashboard production. The budget sales data were present in an Excel spreadsheet.

Below are the scripts created for extracting and cleaning the necessary data. Some columns were left as comments to facilitate insertion into the model for future expansions or report improvements.

DIM_Calendar:
sql
Copy code
-- Cleansed DIM_Date Table --
SELECT 
  [DateKey], 
  [FullDateAlternateKey] AS Date, 
  [EnglishDayNameOfWeek] AS Day, 
  Left([EnglishMonthName], 3) AS MonthShort,
  [MonthNumberOfYear] AS MonthNo, 
  [CalendarQuarter] AS Quarter, 
  [CalendarYear] AS Year
FROM 
 [AdventureWorksDW2019].[dbo].[DimDate]
WHERE 
  CalendarYear >= 2019
DIM_Customers:
sql
Copy code
-- Cleansed DIM_Customers Table --
SELECT 
  c.customerkey AS CustomerKey, 
  c.firstname AS [First Name], 
  c.lastname AS [Last Name], 
  c.firstname + ' ' + lastname AS [Full Name], 
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS Gender,
  c.datefirstpurchase AS DateFirstPurchase, 
  g.city AS [Customer City]
FROM 
  [AdventureWorksDW2019].[dbo].[DimCustomer] as c
  LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey 
ORDER BY 
  CustomerKey ASC
DIM_Products:
sql
Copy code
-- Cleansed DIM_Products Table --
SELECT 
  p.[ProductKey], 
  p.[ProductAlternateKey] AS ProductItemCode, 
  p.[EnglishProductName] AS [Product Name], 
  ps.EnglishProductSubcategoryName AS [Sub Category],
  pc.EnglishProductCategoryName AS [Product Category], 
  p.[Color] AS [Product Color], 
  p.[Size] AS [Product Size], 
  p.[ProductLine] AS [Product Line], 
  p.[ModelName] AS [Product Model Name], 
  p.[EnglishDescription] AS [Product Description], 
  ISNULL (p.Status, 'Outdated') AS [Product Status] 
FROM 
  [AdventureWorksDW2019].[dbo].[DimProduct] as p
  LEFT JOIN dbo.DimProductSubcategory AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey 
  LEFT JOIN dbo.DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey 
ORDER BY 
  p.ProductKey ASC
FACT_InternetSales:
sql
Copy code
-- Cleansed FACT_InternetSales Table --
SELECT 
  [ProductKey], 
  [OrderDateKey], 
  [DueDateKey], 
  [ShipDateKey], 
  [CustomerKey], 
  [SalesOrderNumber], 
  [SalesAmount] 
FROM 
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE 
  LEFT (OrderDateKey, 4) >= YEAR(GETDATE()) - 2
ORDER BY
  OrderDateKey ASC
4 - Data Modeling
Data modeling was performed after the cleaning and importing of tables into Power BI. The figure below shows how the FACT_Budget table was connected to the FACT_InternetSales table, as well as the other Dimension Tables.

Data Model

5 - Sales Management Dashboard
A dashboard containing an overview of Adventure Works' internet sales was delivered as a product. Two additional dashboards were added to provide more information and details about sales by customer and/or product.

Sales Management Dashboards

The Power BI file containing the produced dashboards is available on my GitHub.
