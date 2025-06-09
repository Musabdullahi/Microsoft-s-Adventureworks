# AdventureWorks Data Analysis Project

Welcome to the AdventureWorks Data Analysis Project!

This piece delves into a comprehensive exploration of the AdventureWorks dataset, a rich and versatile sample database provided by Microsoft. with a realistic simulation of a company's sales, product, and customer data. AdeventureWorks serves as an invaluable resource for honing SQL, data analysis and business intelligence skills.

In this project, I leverage SQL to explore and analyse key aspects of AdventureWorks, including customer demographics, sales trends, product performance and geographic patterns. Each analysis is designed to emulate real-world business scenarios and provide actionable insights, reflecting both technical skills and analytical approach.

### key objectives of the project:

*Data Exploration:* Navigating the AdventureWorks database schema to understand the structure and relationships across tables.

*Data Transformation:* Using SQL to clean, join and format data for conmprehensive analysis ensuring accuracy and readability.

*Analytical Insights:* Generating insightful reports that highlights sales trends, customer preferences and product performance, each presented through intuitive data visualisation.

This project demonstrates my proficiency in advanced SQL queries, MySQL Workbench, and PowerBi as data visualisation tool. The analysis techniques here are highly adaptable for any database or Bi Environment.

Feel free to explore the analysis files, SQL scripts and visualisation to see my approach to transforming raw data into meaningful business insights! 



## PROCESSES


```sql
Data Import - Schema Connections and Relationship

-- Consolidating the 3 sales table as 1

	CREATE TABLE consolidated_sales AS

	WITH all_sales AS (
	SELECT * FROM sales_data_2020
	UNION
	SELECT * FROM sales_data_2021
	UNION
	SELECT * FROM sales_data_2022
	)

	SELECT * FROM all_sales;


Data Cleaning For consolidated_sales TABLE

-- Checking for Duplicates

	SELECT OrderDate,StockDate,OrderNumber,ProductKey,CustomerKey,TerritoryKey,OrderLineItem,OrderQuantity, COUNT(*) AS count
	FROM sales_data_2020
	GROUP BY OrderDate,StockDate,OrderNumber,ProductKey,CustomerKey,TerritoryKey,OrderLineItem,OrderQuantity
	HAVING COUNT(*)>1;

-- Checking for Null
	
	SELECT *
	FROM consolidated_sales
	WHERE OrderDate IS NULL OR StockDate IS NULL OR OrderNumber IS NULL OR ProductKey IS NULL OR CustomerKey IS NULL OR 		TerritoryKey IS NULL OR OrderLineItem IS NULL OR OrderQuantity IS NULL;


-- Formatting date for Consistency
	
	SELECT OrderDate, StockDate	
	FROM consolidated_sales;    

	UPDATE consolidated_sales
	SET StockDate = REPLACE (StockDate, '-', '/');
    

	UPDATE consolidated_sales	
	SET StockDate = DATE_FORMAT(STR_TO_DATE(Stockdate, '%d/%m/%Y'), '%Y/%m/%d')
	WHERE CAST(SUBSTRING_INDEX(Stockdate, '/', 1) AS UNSIGNED) > 12
		AND StockDate NOT LIKE '____/__/__';


	UPDATE consolidated_sales	
	SET StockDate = DATE_FORMAT(STR_TO_DATE(StockDate, '%m/%d/%Y'), '%Y/%m/%d')	
	WHERE CAST(SUBSTRING_INDEX(StockDate, '/', 1) AS UNSIGNED) <= 12
		AND StockDate NOT LIKE '____/__/__';

	ALTER TABLE consolidated_sales	
	MODIFY OrderDate DATE;

	ALTER TABLE consolidated_sales	
	MODIFY StockDate DATE;

Data Cleaning For customer_lookup TABLE:

-- Checking for Duplicates

	SELECT DISTINCT count(CustomerKey)	
	FROM customer_lookup;

-- Checking for Null
	
	SELECT *
	FROM customer_lookup
	WHERE CustomerKey IS NULL OR Prefix IS NULL OR FirstName IS NULL OR LastName IS NULL OR BirthDate IS NULL OR MaritalStatus IS 		NULL OR Gender IS NULL;

-- Fixing For NULL

	UPDATE adventureworks.customer_lookup
	SET Prefix = NULL
	where Prefix = '';
	
	UPDATE adventureworks.customer_lookup
	SET Gender = NULL
	where Gender = 'NA';

Other TABLES are clean without Errors

-- Checking For DataTypes
	
	DESCRIBE consolidated_sales;

	DESCRIBE customer_lookup;

	SELECT * FROM customer_lookup;   
    
	ALTER TABLE customer_lookup
	MODIFY BirthDate DATE;
	DESCRIBE product_lookup;
    	
	SELECT * FROM product_lookup;

	ALTER TABLE product_lookup
	MODIFY ProductCost DECIMAL(10,2);
    
	UPDATE product_lookup
	SET ProductCost = ROUND(ProductCost,2);
	
	UPDATE product_lookup
	SET ProductPrice = ROUND(ProductPrice,2);

	UPDATE product_lookup
	SET ProductColor = NULL
	WHERE ProductColor = 'NA';

	UPDATE product_lookup
	SET ProductSize = NULL
	WHERE ProductSize = '0';
	
	ALTER TABLE customer_lookup
	ADD COLUMN FullName varchar(255)
	GENERATED ALWAYS AS (CONCAT_WS(FirstName,' ',LastName));
    

	ALTER TABLE customer_lookup
	ADD COLUMN Age int;
	
	UPDATE customer_lookup
	SET Age = timestampdiff(YEAR, BirthDate, CURDATE());
	
	ALTER TABLE customer_lookup
	ADD COLUMN AgeRange VARCHAR(30);

	UPDATE customer_lookup
	SET AgeRange =
		CASE WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 41 AND 50 THEN '41-50'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 51 AND 60 THEN '51-60'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 61 AND 70 THEN '61-70'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 71 AND 80 THEN '71-80'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 81 AND 90 THEN '81-90'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 91 AND 100 THEN '91-100'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 101 AND 110 THEN '101-110'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 111 AND 120 THEN '111-120' 
		ELSE 'Out of Range'
		END;
    

	SELECT  *
	FROM adventureworks.customer_lookup;

Further Scrutiny Shows That Other Tables are Clean/Error-Free/Consistent


-- ANALYSIS
-- Analysis shows that multiple products were ordered same time indicating same OrderNumber
	SELECT OrderNumber, count(OrderNumber)
	FROM adventureworks.consolidated_sales
	GROUP BY OrderNumber 
	HAVING count(OrderNumber) > 1;

	SELECT *	
	FROM consolidated_sales
	WHERE OrderNumber = 'SO51179';

-- Consolidating and Linking ProductKey with Product Lookup
	SELECT * 
	FROM adventureworks.consolidated_sales AS S
	LEFT JOIN product_lookup AS PL 
	ON S.ProductKey = PL.ProductKey
	WHERE OrderNumber = 'SO51179';

-- Adding Dynamic Columns to Customer Lookup
	ALTER TABLE customer_lookup
	ADD COLUMN FullName varchar(255)
	GENERATED ALWAYS AS (CONCAT_WS(FirstName,' ',LastName));

	ALTER TABLE customer_lookup
	ADD COLUMN Age int;
	UPDATE customer_lookup
		
	SET Age = timestampdiff(YEAR, BirthDate, CURDATE());
	ALTER TABLE customer_lookup
	ADD COLUMN AgeRange VARCHAR(30);

	UPDATE customer_lookup		
	SET AgeRange =
		CASE 	WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 41 AND 50 THEN '41-50'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 51 AND 60 THEN '51-60'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 61 AND 70 THEN '61-70'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 71 AND 80 THEN '71-80'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 81 AND 90 THEN '81-90'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 91 AND 100 THEN '91-100'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 101 AND 110 THEN '101-110'
			WHEN timestampdiff(YEAR, BirthDate, CURDATE()) BETWEEN 111 AND 120 THEN '111-120' 
         	ELSE 'Out of Range'
		END;



	SELECT  *    FROM adventureworks.customer_lookup;

