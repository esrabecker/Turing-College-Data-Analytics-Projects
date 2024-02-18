# Basic SQL Tasks using Bigquery

## 1. An Overview of Products

1.1 You’ve been asked to extract the data on products from the Product table where there exists a product subcategory. And also include the name of the ProductSubcategory.


- Columns needed: ProductId, Name, ProductNumber, size, color, ProductSubcategoryId, Subcategory name.

- Order results by SubCategory name.

```sql

SELECT
  product.ProductId,
  product.Name,
  product.ProductNumber,
  product.size,
  product.color,
  product_subcategory.ProductSubcategoryId,
  product_subcategory.name AS SubCategory
FROM
  tc-da-1.adwentureworks_db.product AS product
JOIN
  tc-da-1.adwentureworks_db.productsubcategory AS product_subcategory
ON
  product.ProductSubcategoryId = product_subcategory.ProductSubcategoryID
ORDER BY
  SubCategory;

```

1.2 In 1.1 query you have a product subcategory but see that you could use the category name.

- Find and add the product category name.
- Afterwards, order the results by Category name.

```sql

SELECT
  product.ProductId,
  product.Name,
  product.ProductNumber,
  product.size,
  product.color,
  product_subcategory.ProductSubcategoryId,
  product_subcategory.name AS SubCategory,
  product_category.Name AS Category
FROM
  tc-da-1.adwentureworks_db.product AS product
JOIN
  tc-da-1.adwentureworks_db.productsubcategory AS product_subcategory
ON
  product.ProductSubcategoryID = product_subcategory.ProductSubcategoryID
JOIN
  tc-da-1.adwentureworks_db.productcategory AS product_category
ON
  product_subcategory.ProductCategoryID = product_category.ProductCategoryID
ORDER BY
  Category;

```

1.3 Use the established query to select the most expensive (price listed over 2000) bikes that are still actively sold (does not have a sales end date)

- Order the results from most to least expensive bike.

```sql

SELECT
  product.ProductId,
  product.Name,
  product.ProductNumber,
  product.size,
  product.color,
  product_subcategory.ProductSubcategoryId,
  product_subcategory.name AS SubCategory,
  product_category.Name AS Category,
  product.ListPrice
FROM
  tc-da-1.adwentureworks_db.product AS product
JOIN
  tc-da-1.adwentureworks_db.productsubcategory AS product_subcategory
ON
  product.ProductSubcategoryID = product_subcategory.ProductSubcategoryID
JOIN
  tc-da-1.adwentureworks_db.productcategory AS product_category
ON
  product_subcategory.ProductCategoryID = product_category.ProductCategoryID
WHERE
  product.ListPrice >2000
  AND product.SellEndDate IS NULL
  AND product_category.Name = 'Bikes'
ORDER BY
  product.ListPrice DESC;

```


## 2. Reviewing work orders

2.1 Create an aggregated query to select the:

Number of unique work orders.
Number of unique products.
Total actual cost.
For each location Id from the 'workoderrouting' table for orders in January 2004.

```sql

SELECT
  workorderrouting.LocationID,
  COUNT (DISTINCT workorderrouting.WorkOrderID) AS no_work_orders,
  COUNT (DISTINCT workorderrouting.ProductID) AS no_unique_products,
  SUM(workorderrouting.ActualCost) AS total_actual_cost
FROM
  `tc-da-1.adwentureworks_db.workorderrouting` AS workorderrouting
WHERE
  workorderrouting.ActualStartDate >= '2004-01-01'
  AND workorderrouting.ActualStartDate <= '2004-01-31'
GROUP BY
  LocationID
ORDER BY
  total_actual_cost DESC;

```

2.2 Update your 2.1 query by adding the name of the location and also add the average days amount between actual start date and actual end date per each location.

```sql

SELECT
  workorderrouting.LocationID,
  location.Name AS Location,
  COUNT (DISTINCT workorderrouting.WorkOrderID) AS no_work_orders,
  COUNT (DISTINCT workorderrouting.ProductID) AS no_unique_products,
  SUM(workorderrouting.ActualCost) AS total_actual_cost,
  ROUND(AVG(DATE_DIFF(ActualEndDate,ActualStartDate,DAY)), 2) AS avg_date_diff
FROM
  `tc-da-1.adwentureworks_db.workorderrouting` AS workorderrouting
JOIN
  `tc-da-1.adwentureworks_db.location` AS location
ON
  workorderrouting.LocationID=location.LocationID
WHERE
  workorderrouting.ActualStartDate >= '2004-01-01'
  AND workorderrouting.ActualStartDate <= '2004-01-31'
GROUP BY
  LocationID,
  Location
ORDER BY
  total_actual_cost DESC;

```

2.3 Select all the expensive work Orders (above 300 actual cost) that happened through January 2004.

```sql

SELECT
  workorderrouting.WorkOrderID,
  SUM(workorderrouting.ActualCost) AS actual_cost
FROM
  `tc-da-1.adwentureworks_db.workorderrouting` AS workorderrouting
WHERE
  workorderrouting.ActualStartDate >= '2004-01-01'
  AND workorderrouting.ActualStartDate <= '2004-01-31'
GROUP BY
  workorderrouting.WorkOrderID
HAVING
  actual_cost > 300;

  ```


## 3. Query validation

3.1 Your colleague has written a query to find the list of orders connected to special offers. The query works fine but the numbers are off, investigate where the potential issue lies.

```sql

SELECT
  sales_detail.SalesOrderId,
  sales_detail.OrderQty,
  sales_detail.UnitPrice,
  sales_detail.LineTotal,
  sales_detail.ProductId,
  sales_detail.SpecialOfferID,
  spec_offer_product.rowguid,
  spec_offer_product.ModifiedDate,
  spec_offer.Category,
  spec_offer.Description
FROM
  tc-da-1.adwentureworks_db.salesorderdetail AS sales_detail
LEFT JOIN
  tc-da-1.adwentureworks_db.specialofferproduct AS spec_offer_product
ON
  sales_detail.productId = spec_offer_product.ProductID
  AND sales_detail.SpecialOfferID=spec_offer_product.SpecialOfferID
LEFT JOIN
  tc-da-1.adwentureworks_db.specialoffer AS spec_offer
ON
  sales_detail.SpecialOfferID = spec_offer.SpecialOfferID
ORDER BY
  LineTotal DESC;

 ```

3.2 Your colleague has written this query to collect basic Vendor information. The query does not work, look into the query and find ways to fix it. Can you provide any feedback on how to make this query be easier to debug/read?

```sql

SELECT
  vendor.VendorId AS Id,
  vendor_contact.ContactId,
  vendor_contact.ContactTypeId,
  vendor.Name,
  vendor.CreditRating,
  vendor.ActiveFlag,
  address.AddressId,
  address.City
FROM
  `tc-da-1.adwentureworks_db.vendor` AS vendor
LEFT JOIN
  `tc-da-1.adwentureworks_db.vendorcontact` AS vendor_contact
ON
  vendor.VendorId = vendor_contact.VendorId
LEFT JOIN
  `tc-da-1.adwentureworks_db.vendoraddress` AS vendoraddress
ON
  vendor.VendorId = vendoraddress.VendorId
LEFT JOIN
  `tc-da-1.adwentureworks_db.address` AS address
ON
  vendoraddress.AddressID = address.AddressID;

```


