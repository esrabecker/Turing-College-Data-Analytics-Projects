# Advanced SQL Tasks Using Bigquery


## 1. Reporting Customer Information

### Task 1.1

You’ve been tasked to create a detailed overview of all individual customers (these are defined by customerType = ‘I’ and/or stored in an individual table). Write a query that provides:

- Identity information: CustomerId, Firstname, Last Name, FullName (First Name & Last Name).
- An Extra column called addressing_title i.e. (Mr. Achong), if the title is missing - Dear Achong.
- Contact information: Email, phone, account number, CustomerType.
- Location information: City, State & Country, address.
- Sales: number of orders, total amount (with Tax), date of the last order.
- Copy only the top 200 rows from your written select ordered by total amount (with tax).

```sql

SELECT
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  CONCAT(contact.FirstName, ' ', contact.LastName) AS FullName,
  CONCAT(IFNULL(contact.Title, 'Dear'), ' ', contact.LastName) AS Addressing_Title,
  contact.EmailAddress AS Email,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  address.City AS City,
  address.AddressLine1 AS AddressLine1,
  stateprovince.Name AS State,
  countryregion.Name AS Country,
  COUNT(DISTINCT soh.SalesOrderID) AS Num_Orders,
  SUM(soh.TotalDue) AS Total_Amount,
  MAX(soh.OrderDate) AS Last_Order_Date
FROM
  `tc-da-1.adwentureworks_db.customer` AS customer

JOIN
  `tc-da-1.adwentureworks_db.salesorderheader` AS soh
ON
  customer.CustomerID= soh.CustomerID 
JOIN
  `tc-da-1.adwentureworks_db.contact` AS contact
ON
  soh.ContactID = contact.ContactId
JOIN
  `tc-da-1.adwentureworks_db.customeraddress` AS caddress
ON
  customer.CustomerID = caddress.CustomerID AND caddress.AddressTypeID = 2
JOIN
  `tc-da-1.adwentureworks_db.address` AS address
ON
  caddress.AddressID = address.AddressID
JOIN
  `tc-da-1.adwentureworks_db.stateprovince` AS stateprovince
ON
  address.StateProvinceID = stateprovince.StateProvinceID
JOIN
  `tc-da-1.adwentureworks_db.countryregion` AS countryregion
ON
  stateprovince.CountryRegionCode = countryregion.CountryRegionCode
WHERE
  customer.CustomerType = 'I'
GROUP BY
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  FullName,
  Addressing_Title,
  contact.EmailAddress,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  City,
  AddressLine1,
  State,
  Country
ORDER BY
  total_amount DESC
LIMIT 200;

```

### Task 1.2

Business finds the original query valuable to analyze customers and now want to get the data from the first query for the top 200 customers with the highest total amount (with tax) who have not ordered for the last 365 days. How would you identify this segment?

Hints:

- You can use temp table, cte and/or subquery of the 1.1 select.
- Note that the database is old and the current date should be defined by finding the latest order date in the orders table.


```sql

SELECT
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  CONCAT(contact.FirstName, ' ', contact.LastName) AS FullName,
  CONCAT(IFNULL(contact.Title, 'Dear'), ' ', contact.LastName) AS Addressing_Title,
  contact.EmailAddress AS Email,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  address.City AS City,
  address.AddressLine1 AS AddressLine1,
  stateprovince.Name AS State,
  countryregion.Name AS Country,
  COUNT(DISTINCT soh.SalesOrderID) AS Num_Orders,
  ROUND(SUM(soh.TotalDue), 3) AS Total_Amount,
  MAX(soh.OrderDate) AS Last_Order_Date
FROM
  `tc-da-1.adwentureworks_db.customer` AS customer

JOIN
  `tc-da-1.adwentureworks_db.salesorderheader` AS soh
ON
  customer.CustomerID= soh.CustomerID 
JOIN
  `tc-da-1.adwentureworks_db.contact` AS contact
ON
  soh.ContactID = contact.ContactId
JOIN
  `tc-da-1.adwentureworks_db.customeraddress` AS caddress
ON
  customer.CustomerID = caddress.CustomerID AND caddress.AddressTypeID = 2
JOIN
  `tc-da-1.adwentureworks_db.address` AS address
ON
  caddress.AddressID = address.AddressID
JOIN
  `tc-da-1.adwentureworks_db.stateprovince` AS stateprovince
ON
  address.StateProvinceID = stateprovince.StateProvinceID
JOIN
  `tc-da-1.adwentureworks_db.countryregion` AS countryregion
ON
  stateprovince.CountryRegionCode = countryregion.CountryRegionCode
WHERE
  customer.CustomerType = 'I'
GROUP BY
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  FullName,
  Addressing_Title,
  contact.EmailAddress,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  City,
  AddressLine1,
  State,
  Country
  HAVING 
  MAX(soh.OrderDate) < DATE_SUB((SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 day)
  ORDER BY
    total_amount DESC
  LIMIT 200 ;

```

### Task 1.3 

Enrich your original 1.1 SELECT by creating a new column in the view that marks active & inactive customers based on whether they have ordered anything during the last 365 days.

- Copy only the top 500 rows from your written select ordered by CustomerId desc.


```sql

SELECT
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  CONCAT(contact.FirstName, ' ', contact.LastName) AS FullName,
  CONCAT(IFNULL(contact.Title, 'Dear'), ' ', contact.LastName) AS Addressing_Title,
  contact.EmailAddress AS Email,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  address.City AS City,
  address.AddressLine1 AS AddressLine1,
  stateprovince.Name AS State,
  countryregion.Name AS Country,
  COUNT(DISTINCT soh.SalesOrderID) AS Num_Orders,
  SUM(soh.TotalDue) AS Total_Amount,
  MAX(soh.OrderDate) AS Last_Order_Date,
  CASE
    WHEN MAX(soh.OrderDate) < DATE_SUB(( SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 day) THEN 'Inactive'
  ELSE
  'Active'
END
  AS Customer_Status
FROM
  `tc-da-1.adwentureworks_db.customer` AS customer
JOIN
  `tc-da-1.adwentureworks_db.salesorderheader` AS soh
ON
  customer.CustomerID= soh.CustomerID
JOIN
  `tc-da-1.adwentureworks_db.contact` AS contact
ON
  soh.ContactID = contact.ContactId
JOIN
  `tc-da-1.adwentureworks_db.customeraddress` AS caddress
ON
  customer.CustomerID = caddress.CustomerID
  AND caddress.AddressTypeID = 2
JOIN
  `tc-da-1.adwentureworks_db.address` AS address
ON
  caddress.AddressID = address.AddressID
JOIN
  `tc-da-1.adwentureworks_db.stateprovince` AS stateprovince
ON
  address.StateProvinceID = stateprovince.StateProvinceID
JOIN
  `tc-da-1.adwentureworks_db.countryregion` AS countryregion
ON
  stateprovince.CountryRegionCode = countryregion.CountryRegionCode
WHERE
  customer.CustomerType = 'I'
GROUP BY
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  FullName,
  Addressing_Title,
  contact.EmailAddress,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  City,
  AddressLine1,
  State,
  Country
ORDER BY
  CustomerID DESC
LIMIT
  500 ;

```


### Task 1.4 

Business would like to extract data on all active customers from North America. Only customers that have either ordered no less than 2500 in total amount (with Tax) or ordered 5 + times should be presented.

- In the output for these customers divide their address line into two columns, i.e.:

- Order the output by country, state, and date_last_order.

```sql

WITH
  customers AS(
  SELECT
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  CONCAT(contact.FirstName, ' ', contact.LastName) AS FullName,
  CONCAT(IFNULL(contact.Title, 'Dear'), ' ', contact.LastName) AS Addressing_Title,
  contact.EmailAddress AS Email,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  address.City AS City,
  address.AddressLine1 AS AddressLine1,
  stateprovince.Name AS State,
  countryregion.Name AS Country,
  COUNT(DISTINCT soh.SalesOrderID) AS Num_Orders,
  SUM(soh.TotalDue) AS Total_Amount,
  MAX(soh.OrderDate) AS Last_Order_Date,
  CASE
    WHEN MAX(soh.OrderDate) < DATE_SUB(( SELECT MAX(OrderDate) FROM `adwentureworks_db.salesorderheader`), INTERVAL 365 day) THEN 'Inactive'
  ELSE
  'Active'
END
  AS Customer_Status
FROM
  `tc-da-1.adwentureworks_db.customer` AS customer
JOIN
  `tc-da-1.adwentureworks_db.salesorderheader` AS soh
ON
  customer.CustomerID= soh.CustomerID
JOIN
  `tc-da-1.adwentureworks_db.contact` AS contact
ON
  soh.ContactID = contact.ContactId
JOIN
  `tc-da-1.adwentureworks_db.customeraddress` AS caddress
ON
  customer.CustomerID = caddress.CustomerID
  AND caddress.AddressTypeID = 2
JOIN
  `tc-da-1.adwentureworks_db.address` AS address
ON
  caddress.AddressID = address.AddressID
JOIN
  `tc-da-1.adwentureworks_db.stateprovince` AS stateprovince
ON
  address.StateProvinceID = stateprovince.StateProvinceID
JOIN
  `tc-da-1.adwentureworks_db.countryregion` AS countryregion
ON
  stateprovince.CountryRegionCode = countryregion.CountryRegionCode
WHERE
  customer.CustomerType = 'I'
GROUP BY
  customer.CustomerID,
  contact.FirstName,
  contact.LastName,
  FullName,
  Addressing_Title,
  contact.EmailAddress,
  contact.Phone,
  soh.AccountNumber,
  customer.CustomerType,
  City,
  AddressLine1,
  State,
  Country
ORDER BY
  CustomerID DESC )
SELECT
  customerID,
  FullName,
  Addressing_Title,
  AddressLine1,
  REGEXP_EXTRACT(customers.AddressLine1, r'^(\d+)') AS Address_No,
  REGEXP_EXTRACT(customers.AddressLine1, r'^(?:\d+ )?(.*)$') AS Address_St,
  City,
  State,
  Country,
  num_orders,
  total_amount,
  last_order_date,
  customer_status
FROM
  customers
WHERE
  (num_orders >= 5
    OR total_amount >= 2500)
  AND customer_status = 'Active'
  AND customers.Country IN('Canada', 'United States')
ORDER BY
  Country,
  customers.State,
  last_order_date;

```


##  2.Reporting Sales numbers

### Task 2.1 

Create a query of monthly sales numbers in each Country & region. Include in the query a number of orders, customers and sales persons in each month with a total amount with tax earned. Sales numbers from all types of customers are required.

- Main tables to start from: salesorderheader.

```sql

SELECT
  LAST_DAY(DATE(salesorderheader.OrderDate)) AS Order_Month,
  salesterritory.countryregioncode AS CountryRegionCode,
  salesterritory.Name AS Region,
  COUNT(DISTINCT salesorderheader.SalesOrderID) AS Number_of_Orders,
  COUNT(DISTINCT salesorderheader.CustomerID) AS Number_of_Customers,
  COUNT(DISTINCT salesorderheader.SalesPersonID) AS Number_of_Salespersons,
  CAST(SUM(salesorderheader.TotalDue) AS INT) AS Total_w_Tax 
FROM
  `adwentureworks_db.salesorderheader` AS salesorderheader
JOIN
  `adwentureworks_db.salesterritory` AS salesterritory
ON
  salesorderheader.TerritoryID = salesterritory.TerritoryID
GROUP BY
  Order_Month,
  CountryRegionCode,
  Region****

```

### Task 2.2

 Enrich 2.1 query with the cumulative_sum of the total amount with tax earned per country & region.

- Hint: use CTE or subquery.

```sql

WITH
  t1 AS (
  SELECT
    DATE_SUB(LAST_DAY(DATE(salesorderheader.OrderDate)), INTERVAL 0 DAY) AS Order_Month,
    salesterritory.countryregioncode AS CountryCode,
    salesterritory.Name AS Region,
    COUNT(DISTINCT salesorderheader.SalesOrderID) AS Number_of_Orders,
    COUNT(DISTINCT salesorderheader.CustomerID) AS Number_of_Customers,
    COUNT(DISTINCT salesorderheader.SalesPersonID) AS Number_of_Salespersons,
    CAST(SUM(salesorderheader.TotalDue) AS INT) AS Total_w_Tax
  FROM
    `adwentureworks_db.salesorderheader` AS salesorderheader
  JOIN
    `adwentureworks_db.salesterritory` AS salesterritory
  ON
    salesorderheader.TerritoryID = salesterritory.TerritoryID
  GROUP BY
    Order_Month,
    CountryCode,
    Region ),
  t2 AS (
  SELECT
    Order_Month,
    t1.CountryCode,
    Region,
    Number_of_Orders,
    Number_of_Customers,
    Number_of_Salespersons,
    Total_w_Tax,
    CAST(SUM(Total_w_Tax) OVER (PARTITION BY t1.CountryCode, t1.Region ORDER BY t1.Order_Month ASC) AS INT ) AS cumulative_sum
  FROM
    `adwentureworks_db.salestaxrate` AS salestr
  JOIN
    `adwentureworks_db.stateprovince` AS statep
  ON
    salestr.StateProvinceID = statep.StateProvinceID
  JOIN
    t1
  ON
    statep.CountryRegionCode = t1.CountryCode
  GROUP BY
    1,
    2,
    3,
    4,
    5,
    6,
    7 )
SELECT
  *
FROM
  t2;

```

### Task 2.3

Enrich 2.2 query by adding ‘sales_rank’ column that ranks rows from best to worst for each country based on total amount with tax earned each month. I.e. the month where the (US, Southwest) region made the highest total amount with tax earned will be ranked 1 for that region and vice versa.


```sql

WITH
  t1 AS (
  SELECT
    DATE_SUB(LAST_DAY(DATE(salesorderheader.OrderDate)), INTERVAL 0 DAY) AS Order_Month,
    salesterritory.countryregioncode AS CountryCode,
    salesterritory.Name AS Region,
    COUNT(DISTINCT salesorderheader.SalesOrderID) AS Number_of_Orders,
    COUNT(DISTINCT salesorderheader.CustomerID) AS Number_of_Customers,
    COUNT(DISTINCT salesorderheader.SalesPersonID) AS Number_of_Salespersons,
    CAST(SUM(salesorderheader.TotalDue) AS INT) AS Total_w_Tax
  FROM
    `adwentureworks_db.salesorderheader` AS salesorderheader
  JOIN
    `adwentureworks_db.salesterritory` AS salesterritory
  ON
    salesorderheader.TerritoryID = salesterritory.TerritoryID
  GROUP BY
    Order_Month,
    CountryCode,
    Region ),
  t2 AS (
  SELECT
    Order_Month,
    t1.CountryCode,
    Region,
    Number_of_Orders,
    Number_of_Customers,
    Number_of_Salespersons,
    Total_w_Tax,
    CAST(SUM(Total_w_Tax) OVER (PARTITION BY t1.CountryCode, t1.Region ORDER BY t1.Order_Month ASC) AS INT ) AS cumulative_sum,
    RANK() OVER(PARTITION BY t1.CountryCode, t1.Region ORDER BY t1.Total_w_Tax DESC) AS sales_rank
  FROM
    `adwentureworks_db.salestaxrate` AS salestr
  JOIN
    `adwentureworks_db.stateprovince` AS statep
  ON
    salestr.StateProvinceID = statep.StateProvinceID
  JOIN
    t1
  ON
    statep.CountryRegionCode = t1.CountryCode
  GROUP BY
    1,
    2,
    3,
    4,
    5,
    6,
    7 )
SELECT
  *
FROM
  t2
--WHERE t2.CountryCode = 'FR'
ORDER BY
  t2.Total_w_Tax DESC;

```

### Task 2.4

Enrich 2.3 query by adding taxes on a country level:

- As taxes can vary in country based on province, the needed column is ‘mean_tax_rate’ -> average tax rate in a country.

- Also, as not all regions have data on taxes, you also want to be transparent and show the ‘perc_provinces_w_tax’ -> a column representing the percentage of provinces with available tax rates for each country (i.e. If US has 53 provinces, and 10 of them have tax rates, then for US it should show 0,19)

- Hint: If a state has multiple tax rates, choose the higher one. Do not double count a state in country average rate calculation if it has multiple tax rates.

- Hint: Ignore the isonlystateprovinceFlag rate mechanic, it is beyond the scope of this exercise. Treat all tax rates as equal.


```sql

WITH
  t1 AS (
  SELECT
    DATE_SUB(LAST_DAY(DATE(salesorderheader.OrderDate)), INTERVAL 0 DAY) AS Order_Month,
    salesterritory.countryregioncode AS CountryCode,
    salesterritory.Name AS Region,
    COUNT(DISTINCT salesorderheader.SalesOrderID) AS Number_of_Orders,
    COUNT(DISTINCT salesorderheader.CustomerID) AS Number_of_Customers,
    COUNT(DISTINCT salesorderheader.SalesPersonID) AS Number_of_Salespersons,
    CAST(SUM(salesorderheader.TotalDue) AS INT) AS Total_w_Tax
  FROM
    `adwentureworks_db.salesorderheader` AS salesorderheader
  JOIN
    `adwentureworks_db.salesterritory` AS salesterritory
  ON
    salesorderheader.TerritoryID = salesterritory.TerritoryID
  GROUP BY
    Order_Month,
    CountryCode,
    Region ),
  t2 AS (
  SELECT
    Order_Month,
    t1.CountryCode,
    Region,
    Number_of_Orders,
    Number_of_Customers,
    Number_of_Salespersons,
    Total_w_Tax,
    RANK() OVER(PARTITION BY t1.CountryCode, t1.Region ORDER BY t1.Total_w_Tax DESC) AS sales_rank,
    CAST(SUM(Total_w_Tax) OVER (PARTITION BY t1.CountryCode, t1.Region ORDER BY t1.Order_Month ASC) AS INT ) AS cumulative_sum,
    ROUND(AVG(salestr.TaxRate), 1) AS mean_tax_rate
  FROM
    `adwentureworks_db.salestaxrate` AS salestr
  JOIN
    `adwentureworks_db.stateprovince` AS statep
  ON
    salestr.StateProvinceID = statep.StateProvinceID
  JOIN
    t1
  ON
    statep.CountryRegionCode = t1.CountryCode
  GROUP BY
    1,
    2,
    3,
    4,
    5,
    6,
    7 ),
  t3 AS(
  SELECT
    CountryRegionCode AS Country,
    ROUND(COUNT (str.StateProvinceID)/ COUNT(sp.StateProvinceID), 2) AS perc_provinces_w_tax,
  FROM
    `adwentureworks_db.stateprovince` AS sp
  LEFT JOIN
    `adwentureworks_db.salestaxrate` AS str
  ON
    sp.StateProvinceID = str.StateProvinceID
  GROUP BY
    Country)
SELECT
  Order_Month,
  CountryCode,
  Region,
  Number_of_Orders,
  Number_of_Customers,
  Number_of_Salespersons,
  Total_w_Tax,
  sales_rank,
  cumulative_sum,
  mean_tax_rate,
  t3.perc_provinces_w_tax
FROM
  t2
JOIN
  t3
ON
  t2.CountryCode = t3.Country

--WHERE CountryCode = 'US'

ORDER BY
  t2.Total_w_Tax DESC;

```
