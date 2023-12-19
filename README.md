Projeto de Análise de Dados - Gerenciamento de Vendas
-----------------------------------------------------

![](https://phscala.files.wordpress.com/2021/07/sales-overview-1.png)

![](https://phscala.files.wordpress.com/2021/07/data_model-2.png)

![](https://phscala.files.wordpress.com/2021/07/dim-customer-csv-excel-2.png)

![](https://phscala.files.wordpress.com/2021/07/sql-calendar-project-1-2.png)

1 - Introdução & Setup
----------------------

Adventure Works é uma empresa fictícia. A AW é uma grande empresa de manufatura multinacional que produz e distribui bicicletas, peças de bicicleta e acessórios. Comercializa nos mercados Norte Americano, Europeu e Asiático. A empresa possui 500 funcionários. A AW possui diversos grupos de vendas.

O link para download do Data Warehouse da AW se encontra abaixo.

**<https://docs.microsoft.com/pt-br/sql/samples/adventureworks-install-configure?view=sql-server-ver15&tabs=ssms>**

Para a realização do processo de ETL foi utilizado o SQL Server. Para a visualização de dados foi utilizado o Power BI. O DB foi atualizado pela última vez em 2014. Para atualizar o DB foi utilizado um script em SQL disponível no meu [github.](https://github.com/PedroScala/Projeto-de-Gerenciamento-de-Vendas-AW)

2 - Requisitos de Negócios e Necessidades do Cliente
----------------------------------------------------

Os requisitos de negócio e necessidade do cliente foram estabelecidas pelo Gerente de Vendas. Baseado nas exigências do colaborador foi gerada a tabela abaixo que contêm os critérios de aceitação e detalhes para produção do painel.

| No # | Cargo                   | Demanda                                                | Objetivo                                                  | Critério de Aceitação                 |
|-----|-------------------------|--------------------------------------------------------|-----------------------------------------------------------|--------------------------------------|
| 1   | Gerente de Vendas        | Produzir painel com visão geral das vendas pela internet | Melhorar o acompanhamento sobre os melhores clientes e produtos | Um relatório que se atualiza diariamente |
| 2   | Representante de Vendas | Visão detalhada das vendas pela internet por cliente    | Monitorar os clientes que mais compram e para quem é possível vender mais | Um painel que permita filtrar por cliente |
| 3   | Representante de Vendas | Visão detalhada das vendas pela internet por produto    | Monitorar os produtos com alto desempenho em vendas        | Um painel que permita filtrar por produto (categoria e subcategoria) |
| 4   | Gerente de Vendas        | Produzir painel com visão geral das vendas pela internet | Monitorar as vendas e orçamento ao longo do tempo         | Um gráfico e um KPI que compare as vendas e orçamento ao longo do tempo |


Tabela 2.1 - Requisitos de Negócio e do Cliente

3 - Limpeza e Transformação dos Dados (SQL)
-------------------------------------------

Nessa etapa utilizei o SQL Server para acessar o DW e obter os dados necessarios para a produção do painel. Lembrando que os dados referente ao orçamento de venda estavam presente em uma planilha de Excel.

Abaixo é possível verificar os scripts criados para a extração e limpeza dos dados necessários. Algumas colunas foram deixadas como comentários para facilitar a inserção no modelo em futuras expansões ou melhoria nos relatórios.

* * * * *

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

4 - Modelagem dos Dados
-----------------------

Foi realizada a modelagem dos dados após a limpeza e importação das tabelas para o PowerBI. Na figura abaixo é possível verificar como a tabela FACT_Budget foi conectada a tabela FACT_InternetSales assim como as demais Tabelas Dimensão.

![](https://phscala.files.wordpress.com/2021/07/data_model-3.png?w=1024)

Figura 4.1 - Modelo dos Dados

5 - Painel de Gerenciamento de Vendas
-------------------------------------

Como produto foi entregue uma Painel contendo uma visão geral das vendas via internet da Adventure Works. Foram adicionados outros dois painéis para fornecer mais informações e detalhes sobre as vendas por cliente e/ou produto.

![](https://phscala.files.wordpress.com/2021/07/sales-overview-2.png?w=1024)

Figura 5.1 - Painéis para Gerenciamento das Vendas Online

![](https://phscala.files.wordpress.com/2021/07/customer_details.png?w=1024)

![](https://phscala.files.wordpress.com/2021/07/porduct_details.png?w=1024)

O arquivo do Power BI contendo os painéis produzidos está disponível no meu [Github.](https://github.com/PedroScala/Projeto-de-Gerenciamento-de-Vendas-AW)
