# Database Normalization 
As part of the preprocessing steps, we undertook database normalization to optimize the dataset structure. Database normalization is the process of organizing a database schema to reduce redundancy and dependency, thereby improving data integrity, and minimizing anomalies. By splitting the dataset into multiple relational tables and establishing relationships between them, we effectively normalized the data structure, facilitating efficient data retrieval, analysis, and maintenance.
## Tables Created
The following were the tables created.

1. Departments Table: The Departments table represents distinct departments within the organization. Each department is uniquely identified by a Department_Id and is associated with a Department_Name.
2. Categories Table: the Categories table encompasses different product categories available in the inventory. Each category is identified by a Category_Id and is associated with a Category_Name.
3. Customers Table:The Customers table stores information about individual customers. It includes details such as Customer_Id, First_Name, Last_Name, Segment, Street, City, State, and Country. Each customer is uniquely identified by a Customer_Id.
4. Products Table:The Products table catalogs various products offered by the organization. It includes Product_Id, Product_Name, Price, Status, and Category_Id, where each product is associated with a specific category.
5. Orders Table:The Orders table captures essential details regarding customer orders. It includes Order_Id, Customer_Id, Order_Date, Shipping_Date, Real_Shipping_Days, Scheduled_Shipping_Days, Delivery_Status, Late_Delivery_Risk, Payment_Type, Market, Benefit_Per_Order, Sales_Per_Customer, Department_Id, Shipping_Mode, Region, Order_Status, Sales_Amount, and Profit.
6. Order_Items Table:The Order_Items table provides a granular view of individual items within each order. It includes Order_Item_Id, Order_Id, Product_Id, Discount_Amount, Discount_Rate, Quantity, Product_Price, Profit_Ratio, and Total_Amount.

### SQL code to create the tables and insert/update the tables
```sql
CREATE TABLE Departments (
    Department_Id INT PRIMARY KEY,
    Department_Name VARCHAR(255)
);

CREATE TABLE Categories (
    Category_Id INT PRIMARY KEY,
    Category_Name VARCHAR(255)
);

CREATE TABLE Customers (
    Customer_Id INT PRIMARY KEY,
    First_Name VARCHAR(255),
    Last_Name VARCHAR(255),
    Segment VARCHAR(255),
    Street VARCHAR(255),
    City VARCHAR(255),
    State VARCHAR(255),
    Country VARCHAR(255)
);

CREATE TABLE Products (
    Product_Id INT PRIMARY KEY,
    Product_Name VARCHAR(255),
    Price DECIMAL(10, 2),
    Status VARCHAR(255),
    Category_Id INT REFERENCES Categories(Category_Id)
);

CREATE TABLE Orders (
    	Order_Id INT PRIMARY KEY,
	Customer_Id INT REFERENCES Customers(Customer_Id),
    	Order_Date DATE,
    	Shipping_Date DATE,
    	Real_Shipping_Days INT,
    	Scheduled_Shipping_Days INT,
	Delivery_Status VARCHAR(255),
    	Late_Delivery_Risk VARCHAR(255),
    	Payment_Type VARCHAR(255),
	Market VARCHAR(255),
    	Benefit_Per_Order DECIMAL(10, 2), 
    	Sales_Per_Customer DECIMAL(10, 2),
    	Department_Id INT REFERENCES Departments(Department_Id),
	Shipping_Mode VARCHAR(50),
 	Region VARCHAR(150),
	Order_Status VARCHAR(50),
	Sales_Amount DECIMAL (10,2),
	Profit DECIMAL(10,2)
);



CREATE TABLE Order_Items (
    Order_Item_Id INT PRIMARY KEY,
    Order_Id INT REFERENCES Orders(Order_Id),
    Product_Id INT REFERENCES Products(Product_Id),
    Discount_Amount DECIMAL(10, 2),
    Discount_Rate DECIMAL(10, 2),
    Quantity INT,
    Product_Price DECIMAL(10, 2),
    Profit_Ratio DECIMAL(10, 2),
    Total_Amount DECIMAL(10, 2)
);


---INSERTING VALUES INTO THE TABLES
INSERT INTO Categories(Category_Id)
SELECT DISTINCT Category_Id FROM Main_Table;

-- Update  additional information into Categories table

UPDATE Categories 
SET Category_Name = Main_Table.Category_Name
FROM Categories
JOIN Main_Table ON Categories.Category_Id = Main_Table.Category_Id;

---Insert primary keys into Departments table	
INSERT INTO Departments (Department_Id)
SELECT DISTINCT Department_Id
FROM Main_Table;

-- Update or insert additional information into Departments table
UPDATE Departments 
SET 
    Department_Name = (SELECT DISTINCT Department_Name FROM Main_Table WHERE Departments.Department_Id = Main_Table.Department_Id);

-- insert the customer id into the customers table
INSERT INTO Customers(Customer_Id)
SELECT DISTINCT Customer_Id
FROM Main_Table;

-- Update additional information into Customers table
UPDATE Customers 
SET 
    First_Name = (SELECT DISTINCT Customer_Fname FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    Last_Name = (SELECT DISTINCT Customer_Lname FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    Segment = (SELECT  DISTINCT Customer_Segment FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    Street = (SELECT  DISTINCT Customer_Street FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    City = (SELECT  DISTINCT Customer_City FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    State = (SELECT DISTINCT Customer_State FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id),
    Country = (SELECT DISTINCT Customer_Country FROM Main_Table WHERE Customers.Customer_Id = Main_Table.Customer_Id);

---Inserting the product id into the product table
INSERT INTO Products(Product_Id)
SELECT DISTINCT Product_Card_Id
FROM Main_Table;

-- Update additional information into Products table
UPDATE Products 
SET 
    Product_Name = (SELECT  DISTINCT Product_Name FROM Main_Table WHERE Products.Product_Id = Main_Table.Product_Card_Id),
    Price = (SELECT  DISTINCT Product_Price FROM Main_Table WHERE Products.Product_Id = Main_Table.Product_Card_Id),
    Status = (SELECT DISTINCT Product_Status FROM Main_Table WHERE Products.Product_Id = Main_Table.Product_Card_Id),
    Category_Id = (SELECT DISTINCT Product_Category_Id FROM Main_Table WHERE Products.Product_Id = Main_Table.Product_Card_Id);


-- Insert primary keys into orders table
-- My choice of unique id for each order, stems from the fact that, same order ID is assigned to multiple orders from the same customer.
-- After verifying that there are no dupicates, and the order_item_Id also uniquely identifies individual order. I concluded to use the Order_Item_Id.
-- Note this is a practise project and this approach is not ideal in the real world unless, clients or stakeholders accepts such approach
INSERT INTO Orders(Order_Id)
SELECT DISTINCT Order_Item_Id FROM Main_Table; --- Inserting distinct order_item_id values from Main_Table into the Orders table.
												-- Using order_item_id as the order_id because the order_id alone is not unique for each order due to
												-- multiple products being purchased in a single order. The order_item_id uniquely identifies each order item,
												-- ensuring that each row in the Orders table represents a distinct order item, despite potential repetitions in the order_id.

--insert additional information into Orders table
UPDATE Orders 
SET 
    Customer_Id = (SELECT DISTINCT Order_Customer_Id FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Order_Date = (SELECT DISTINCT order_date_DateOrders FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Shipping_Date = (SELECT DISTINCT shipping_date_DateOrders FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Real_Shipping_Days = (SELECT  DISTINCT Days_for_shipping_real FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Scheduled_Shipping_Days = (SELECT DISTINCT  Days_for_shipment_scheduled FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Delivery_Status = (SELECT DISTINCT Delivery_Status FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Late_Delivery_Risk = (SELECT DISTINCT Late_delivery_risk FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Payment_Type = (SELECT DISTINCT Type FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
    Market = (SELECT DISTINCT Market FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Benefit_Per_Order = (SELECT DISTINCT Benefit_per_order FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id), 
    Sales_Per_Customer = (SELECT DISTINCT Sales_per_customer FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Department_Id = (SELECT DISTINCT Department_Id FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Shipping_mode = (SELECT DISTINCT Shipping_Mode FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Region = (SELECT DISTINCT Order_Region FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Order_Status = (SELECT DISTINCT Order_Status FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Sales_Amount = (SELECT DISTINCT Sales FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id),
	Profit = (SELECT DISTINCT Order_Profit_Per_Order FROM Main_Table WHERE Orders.Order_Id = Main_Table.Order_Item_Id) ; 


SELECT  DISTINCT TOP 5  Order_item_id, Order_Customer_Id,  order_date_DateOrders, shipping_date_DateOrders,Delivery_Status,Late_delivery_risk,Type,Market,Benefit_per_order,Sales_per_customer, Department_Id
FROM Main_Table
ORDER BY Order_Item_Id ASC


INSERT INTO Order_Items(Order_Item_Id)
SELECT DISTINCT Order_Item_Id
FROM Main_Table;

-- Update  additional information into Order_Items table
UPDATE Order_Items 
SET 
    Order_Id = (SELECT DISTINCT Order_Id FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Product_Id = (SELECT DISTINCT Order_Item_Cardprod_Id FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Discount_Amount = (SELECT DISTINCT Order_Item_Discount FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Discount_Rate = (SELECT DISTINCT Order_Item_Discount_Rate FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Quantity = (SELECT DISTINCT Order_Item_Quantity FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Product_Price = (SELECT DISTINCT Order_Item_Product_Price FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Profit_Ratio = (SELECT DISTINCT Order_Item_Profit_Ratio FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Sales_Amount = (SELECT DISTINCT Sales FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id),
    Total_Amount = (SELECT DISTINCT Order_Item_Total FROM Main_Table WHERE Order_Items.Order_Item_Id = Main_Table.Order_Item_Id);



