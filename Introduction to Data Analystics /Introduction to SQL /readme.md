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
