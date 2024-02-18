# Advanced SQL Tasks Using Bigquery

## Task 1.1

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

## Task 1.2

Business finds the original query valuable to analyze customers and now want to get the data from the first query for the top 200 customers with the highest total amount (with tax) who have not ordered for the last 365 days. How would you identify this segment?

Hints:

- You can use temp table, cte and/or subquery of the 1.1 select.
- Note that the database is old and the current date should be defined by finding the latest order date in the orders table.
