Data Analysis Project - Sales Management
-----------------------------------------------------

![Sales Overview](https://phscala.files.wordpress.com/2021/07/sales-overview-1.png?w=400) ![Data Model](https://phscala.files.wordpress.com/2021/07/data_model-2.png?w=400)

![Dim Customer CSV Excel](https://phscala.files.wordpress.com/2021/07/dim-customer-csv-excel-2.png?w=400) ![SQL Calendar Project](https://phscala.files.wordpress.com/2021/07/sql-calendar-project-1-2.png?w=400)


1 - Introduction & Setup
----------------------

Adventure Works is a fictional company. AW is a large multinational manufacturing company that produces and distributes bicycles, bike parts, and accessories. It operates in the North American, European, and Asian markets. The company has 500 employees. AW has various sales groups.

The link to download the AW Data Warehouse is below.

**<https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=ssms>**

For the ETL process, SQL Server was used. Power BI was used for data visualization. The database was last updated in 2014. To update the database, a SQL script available on [my GitHub](https://github.com/PedroScala/Projeto-de-Gerenciamento-de-Vendas-AW) was used.

## 2 - Business Requirements and Customer Needs

Business requirements and customer needs were established by the Sales Manager. Based on the employee's requirements, the table below was generated, containing acceptance criteria and details for panel production.

| No # | Position                | Demand                                                 | Objective                                                 | Acceptance Criteria                  |
|-----|-------------------------|--------------------------------------------------------|-----------------------------------------------------------|--------------------------------------|
| 1   | Sales Manager            | Produce a dashboard with an overview of internet sales  | Improve tracking of the best customers and products        | A report that updates daily          |
| 2   | Sales Representative     | Detailed view of internet sales per customer            | Monitor the customers who buy the most and identify potential for more sales | A dashboard that allows filtering by customer |
| 3   | Sales Representative     | Detailed view of internet sales per product             | Monitor products with high sales performance               | A dashboard that allows filtering by product (category and subcategory) |
| 4   | Sales Manager            | Produce a dashboard with an overview of internet sales  | Monitor sales and budget over time                          | A chart and a KPI that compare sales and budget over time |

**Table 2.1 - Business and Customer Requirements**

## 3 - Data Cleaning and Transformation (SQL)

In this stage, SQL Server was used to access the data warehouse and obtain the necessary data for panel production. Note that the sales budget data was present in an Excel spreadsheet.

Below, you can see the scripts created for extracting and cleaning the necessary data. Some columns were left as comments to facilitate insertion into the model for future expansions or improvements in reports.


**DIM_Calendar:**
-----------------

```
-- Cleansed DIM_Date Table --
SELECT
  [DateKey],
  [FullDateAlternateKey] AS Date,
  --[DayNumberOfWeek],
  [EnglishDayNameOfWeek] AS Day,
  --[SpanishDayNameOfWeek],
  --[FrenchDayNameOfWeek],
  --[DayNumberOfMonth],
  --[DayNumberOfYear],
  --[WeekNumberOfYear],
  [EnglishMonthName] AS Month,
  Left([EnglishMonthName], 3) AS MonthShort,   -- Useful for front end date navigation and front end graphs.
  --[SpanishMonthName],
  --[FrenchMonthName],
  [MonthNumberOfYear] AS MonthNo,
  [CalendarQuarter] AS Quarter,
  [CalendarYear] AS Year --[CalendarSemester],
  --[FiscalQuarter],
  --[FiscalYear],
  --[FiscalSemester]
FROM
 [AdventureWorksDW2019].[dbo].[DimDate]
WHERE
  CalendarYear >= 2019
```

**DIM_Customers**:
------------------

```
-- Cleansed DIM_Customers Table --
SELECT
  c.customerkey AS CustomerKey,
  --      ,[GeographyKey]
  --      ,[CustomerAlternateKey]
  --      ,[Title]
  c.firstname AS [First Name],
  --      ,[MiddleName]
  c.lastname AS [Last Name],
  c.firstname + ' ' + lastname AS [Full Name],
  -- Combined First and Last Name
  --      ,[NameStyle]
  --      ,[BirthDate]
  --      ,[MaritalStatus]
  --      ,[Suffix]
  CASE c.gender WHEN 'M' THEN 'Male' WHEN 'F' THEN 'Female' END AS Gender,
  --      ,[EmailAddress]
  --      ,[YearlyIncome]
  --      ,[TotalChildren]
  --      ,[NumberChildrenAtHome]
  --      ,[EnglishEducation]
  --      ,[SpanishEducation]
  --      ,[FrenchEducation]
  --      ,[EnglishOccupation]
  --      ,[SpanishOccupation]
  --      ,[FrenchOccupation]
  --      ,[HouseOwnerFlag]
  --      ,[NumberCarsOwned]
  --      ,[AddressLine1]
  --      ,[AddressLine2]
  --      ,[Phone]
  c.datefirstpurchase AS DateFirstPurchase,
  --      ,[CommuteDistance]
  g.city AS [Customer City] -- Joined in Customer City from Geography Table
FROM
  [AdventureWorksDW2019].[dbo].[DimCustomer] as c
  LEFT JOIN dbo.dimgeography AS g ON g.geographykey = c.geographykey
ORDER BY
  CustomerKey ASC -- Ordered List by CustomerKey
```

**DIM_Products:**
-----------------

```
-- Cleansed DIM_Products Table --
SELECT
  p.[ProductKey],
  p.[ProductAlternateKey] AS ProductItemCode,
  --      ,[ProductSubcategoryKey],
  --      ,[WeightUnitMeasureCode]
  --      ,[SizeUnitMeasureCode]
  p.[EnglishProductName] AS [Product Name],
  ps.EnglishProductSubcategoryName AS [Sub Category], -- Joined in from Sub Category Table
  pc.EnglishProductCategoryName AS [Product Category], -- Joined in from Category Table
  --      ,[SpanishProductName]
  --      ,[FrenchProductName]
  --      ,[StandardCost]
  --      ,[FinishedGoodsFlag]
  p.[Color] AS [Product Color],
  --      ,[SafetyStockLevel]
  --      ,[ReorderPoint]
  --      ,[ListPrice]
  p.[Size] AS [Product Size],
  --      ,[SizeRange]
  --      ,[Weight]
  --      ,[DaysToManufacture]
  p.[ProductLine] AS [Product Line],
  --     ,[DealerPrice]
  --      ,[Class]
  --      ,[Style]
  p.[ModelName] AS [Product Model Name],
  --      ,[LargePhoto]
  p.[EnglishDescription] AS [Product Description],
  --      ,[FrenchDescription]
  --      ,[ChineseDescription]
  --      ,[ArabicDescription]
  --      ,[HebrewDescription]
  --      ,[ThaiDescription]
  --      ,[GermanDescription]
  --      ,[JapaneseDescription]
  --      ,[TurkishDescription]
  --      ,[StartDate],
  --      ,[EndDate],
  ISNULL (p.Status, 'Outdated') AS [Product Status]
FROM
  [AdventureWorksDW2019].[dbo].[DimProduct] as p
  LEFT JOIN dbo.DimProductSubcategory AS ps ON ps.ProductSubcategoryKey = p.ProductSubcategoryKey
  LEFT JOIN dbo.DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
order by
  p.ProductKey asc
```

**FACT_InternetSales:**
-----------------------

```
-- Cleansed FACT_InternetSales Table --
SELECT
  [ProductKey],
  [OrderDateKey],
  [DueDateKey],
  [ShipDateKey],
  [CustomerKey],
  --  ,[PromotionKey]
  --  ,[CurrencyKey]
  --  ,[SalesTerritoryKey]
  [SalesOrderNumber],
  --  [SalesOrderLineNumber],
  --  ,[RevisionNumber]
  --  ,[OrderQuantity],
  --  ,[UnitPrice],
  --  ,[ExtendedAmount]
  --  ,[UnitPriceDiscountPct]
  --  ,[DiscountAmount]
  --  ,[ProductStandardCost]
  --  ,[TotalProductCost]
  [SalesAmount] --  ,[TaxAmt]
  --  ,[Freight]
  --  ,[CarrierTrackingNumber]
  --  ,[CustomerPONumber]
  --  ,[OrderDate]
  --  ,[DueDate]
  --  ,[ShipDate]
FROM
  [AdventureWorksDW2019].[dbo].[FactInternetSales]
WHERE
  LEFT (OrderDateKey, 4) >= YEAR(GETDATE()) -2 -- Ensures we always only bring two years of date from extraction.
ORDER BY
  OrderDateKey ASC
```

## 4 - Data Modeling

Data modeling was performed after cleaning and importing the tables into PowerBI. In the figure below, you can see how the FACT_Budget table was connected to the FACT_InternetSales table, as well as the other Dimension Tables.

![Data Model](https://phscala.files.wordpress.com/2021/07/data_model-3.png?w=1024)

**Figure 4.1 - Data Model**

## 5 - Sales Management Dashboard

A dashboard containing an overview of Adventure Works internet sales was delivered as a product. Two additional dashboards were added to provide more information and details about sales by customer and/or product.

![Sales Overview](https://phscala.files.wordpress.com/2021/07/sales-overview-2.png?w=1024)

**Figure 5.1 - Dashboards for Online Sales Management**

![Customer Details](https://phscala.files.wordpress.com/2021/07/customer_details.png?w=1024)

![Product Details](https://phscala.files.wordpress.com/2021/07/porduct_details.png?w=1024)

The Power BI file containing the produced dashboards is available on [my GitHub.](https://github.com/PedroScala/Projeto-de-Gerenciamento-de-Vendas-AW)
