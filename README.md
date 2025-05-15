# **Amazon USA Sales Analysis Project**

### **Difficulty Level: Advanced**

## **Project Overview**

I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behavior, product performance, and sales trends using PostgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

The project also focuses on data cleaning, handling null values, and solving real-world business problems using structured queries.

An ERD diagram is included to visually represent the database schema and relationships between tables.

## **Database Setup & Design**

### **Schema Structure**
```sql
-- category table
create table category(
	category_id int primary key,
	category_name varchar(25)
)
-- customers table
create table customers(
	customer_id int primary key,
	first_name varchar(20),
	last_name varchar(20),
	state varchar(20),
	address varchar(5) default ('xxx')
)

-- sellers table
create table sellers(
	seller_id int primary key,
	seller_name varchar(25),
	origin varchar(10)
)


-- products table
create table products(
	product_id int primary key,
	product_name varchar(50),
	price numeric,
	cogs numeric,
	category_id int, --fk
	constraint products_fk_category foreign key (category_id) references category(category_id)
	
)

-- orders table
create table orders(
	order_id int primary key,
	order_date date,
	customer_id int, --fk
	seller_id int, --fk
	order_status varchar(15),
	constraint fk_orders_customer foreign key (customer_id) references customers(customer_id),
	constraint fk_orders_seller foreign key (seller_id) references sellers(seller_id)
)


-- order_items table

create table order_items(
	order_item_id int primary key,
	order_id int, --fk
	product_id int, --fk
	quantity int,
	price_per_unit numeric,
	constraint fk_order_items_order foreign key (order_id) references orders(order_id),
	constraint fk_order_items_product foreign key (product_id) references products(product_id)
)	


-- payments table 

create table payments(
	payment_id int primary key,
	order_id int, --fk
	payment_date date,
	payment_status varchar(20),
	constraint payments_fk_order foreign key (order_id) references orders(order_id)
)

-- shipping table
create table shipping(
	shipping_id int primary key,
	order_id int, --fk
	shipping_date date,
	return_date date,
	shipping_providers varchar(15),
	delivery_status varchar(15),
	constraint shipping_fk_order foreign key (order_id) references orders(order_id)
)


-- inventory table

create table inventory(
	inventory_id int primary key,
	product_id int, --fk
	stock int,
	warehouse_id int,
	last_stock_date date,
	constraint inventory_fk_order foreign key (product_id) references products(product_id)
)
```

## **Task: Data Cleaning**

I cleaned the dataset by:
- **Removing duplicates**: Duplicates in the customer and order tables were identified and removed.
- **Handling missing values**: Null values in critical fields (e.g., customer address, payment status) were either filled with default values or handled using appropriate methods.

---

## **Handling Null Values**

Null values were handled based on their context:
- **Customer addresses**: Missing addresses were assigned default placeholder values.
- **Payment statuses**: Orders with null payment statuses were categorized as “Pending.”
- **Shipping information**: Null return dates were left as is, as not all shipments are returned.

---

## **Objective**

The primary objective of this project is to showcase SQL proficiency through complex queries that address real-world e-commerce business challenges. The analysis covers various aspects of e-commerce operations, including:
- Customer behavior
- Sales trends
- Inventory management
- Payment and shipping analysis
- Forecasting and product performance
  

## **Identifying Business Problems**

Key business problems identified:
1. Low product availability due to inconsistent restocking.
2. High return rates for specific product categories.
3. Significant delays in shipments and inconsistencies in delivery times.
4. High customer acquisition costs with a low customer retention rate.

---

## **Solving Business Problems**

### Solutions Implemented:
-- Top Selling Products
-- Query the top 10 products by total sales value.
-- Challenge: Include product name, total quantity sold, and total sales value.



```sql
with t1 as(
select
	ot.product_id,
	p.product_name,
	sum(ot.quantity) as total_quantity_sold,
	sum(ot.total_sale) as total_sales_value,
	dense_rank() over(order by sum(ot.total_sale) desc) as rn
from order_items as ot
join products as p
on ot.product_id = p.product_id
group by 1,2)
select
	product_id,
	product_name,
	total_quantity_sold,
	total_sales_value
from t1
where rn <= 10
```




















