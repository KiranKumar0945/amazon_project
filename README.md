# **E-commerce Sales Analysis Project**

### **Difficulty Level: Advanced**

## **Project Overview**

I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behavior, product performance, and sales trends using PostgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

The project also focuses on data cleaning, handling null values, and solving real-world business problems using structured queries.

An ERD diagram is included to visually represent the database schema and relationships between tables.
![image alt](https://github.com/KiranKumar0945/amazon_project/blob/948da97e6fb4c044b9d9347499d71779357a4bad/az_erd.png)

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


### Top Selling Products
### 1. Query the top 10 products by total sales value. 
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


### Revenue by Category
### 2. Calculate total revenue generated by each product category. 
-- Challenge: Include the percentage contribution of each category to total revenue.

```sql
select
	p.category_id,
	c.category_name,
	sum(total_sale) as total_revenue,
	round(sum(total_sale)/(select sum(total_sale) from order_items),2)*100 as percentage_contribution
from order_items as ot
join products as p
on ot.product_id = p.product_id
join category as c
on c.category_id =p.category_id
group by 1,2
order by 1
```

### Average Order Value (AOV)
### 3. Compute the average order value for each customer.
-- Challenge: Include only customers with more than 5 orders.
```sql
select
	o.customer_id,
	concat(c.first_name, ' ', c.last_name) as full_name,
	avg(total_sale) as aov
from order_items as ot
join orders as o 
on ot.order_id = o.order_id
join customers as c 
on o.customer_id = c.customer_id
group by 1,2
having count(o.order_id) > 5
```

### Monthly Sales Trend
###  4.Query monthly total sales over the past year.
-- Challenge: Display the sales trend, grouping by month, return current_month sale, last month sale 
```sql
select
	c.category_name,
	extract(month from order_date),
	extract(year from order_date) ,
	sum(total_sale) as cr_mnth_sale,
	lag(sum(total_sale), 1, null) over(partition by c.category_name order by extract(year from order_date),extract(month from order_date) asc )
from orders as o 
join order_items as ot
on o.order_id = ot.order_id
join products as  p
on ot.product_id = p.product_id
join category as c
on c.category_id = p.category_id
where extract(year from order_date) = 2023 -- where order_date > current_date - interval '1 year' for the past 1 year
group by 1,2,3
```


### Customers with No Purchases
### 5.Find customers who have registered but never placed an order.
-- Challenge: List customer details and the time since their registration.

```sql
-- Approach 1
select
	c.customer_id
	-- reg_date - current_date will give the time since their registration
from customers as c
left join orders as o 
on c.customer_id = o.customer_id
where o.order_id is null

-- Approach 2

select
	customer_id
from customers as c 
where c.customer_id not in (select  distinct customer_id from orders)
```
### Least-Selling Categories by State
### 6. Identify the least-selling product category for each state.
-- Challenge: Include the total sales for that category within each state.
```sql
select
*
from (select 
	cs.state,
	c.category_id,
	c.category_name,
	sum(ot.total_sale) as total_revenue,
	rank() over(partition by cs.state order by sum(ot.total_sale) asc) as rn
from order_items as ot
join products as p
on ot.product_id = p.product_id
join category as c
on c.category_id = p.category_id
join orders as o 
on ot.order_id = o.order_id 
join customers as cs
on cs.customer_id = o.customer_id
group by 1,2,3
order by 1) t1
where rn = 1
```

### 7. Customer Lifetime Value (CLTV)
### Calculate the total value of orders placed by each customer over their lifetime.
-- Challenge: Rank customers based on their CLTV.
```sql
select
	c.customer_id,
	concat(c.first_name, ' ', c.last_name) as full_name,
	sum(total_sale) as cltv,
	rank() over(order by sum(total_sale) desc) as rn
from order_items as ot
join orders as o 
on ot.order_id = o.order_id
join customers as c 
on o.customer_id = c.customer_id
group by 1
```

### 8. Inventory Stock Alerts
### Query products with stock levels below a certain threshold (e.g., less than 10 units).
-- Challenge: Include last restock date and warehouse information.
```sql
select
	i.inventory_id,
	p.product_name,
	i.warehouse_id,
	last_stock_date
from inventory as i 
join products as p
on i.product_id = p.product_id
where stock < 10
```

### Shipping Delays
### 9. Identify orders where the shipping date is later than 3 days after the order date.
-- Challenge: Include customer, order details, and delivery provider.
```sql
select
	o.customer_id,
	o.order_id,
	ot.product_id,
	s.shipping_providers,
	(shipping_date - order_date) as time_lapsed
	
from orders as o
join shipping as s
on o.order_id = s.order_id
join order_items as ot
on ot.order_id = o.order_id
where (shipping_date - order_date) > 3
```

### Payment Success Rate 
### 10. Calculate the percentage of successful payments across all orders.
-- Challenge: Include breakdowns by payment status (e.g., failed, pending).
```sql
select
	p.payment_status,
	count(*),
	round(count(*)::numeric/(select count(*) from payments)::numeric * 100,2) 
from orders as o
join payments as p
on o.order_id = p.order_id
group by 1
```
### Top Performing Sellers
### 11. Find the top 5 sellers based on total sales value.
-- Challenge: Include both successful and failed orders, and display their percentage of successful orders.
```sql
with top5 as(
select
	s.seller_id,
	s.seller_name,
	sum(total_sale) as tsv
from order_items as ot
join orders as o 
on ot.order_id = o.order_id
join sellers as s
on o.seller_id = s.seller_id
group by 1,2
order by 3 desc
limit 5),
order_status as(
select
	seller_id,
	order_status,
	count(*) as s_count
from orders 
where order_status not in ('Inprogress', 'Returned')
group by 1,2)
select
	t.seller_id,
	t.seller_name,
	max(case  when order_status = 'Cancelled' then s_count end) as Cancelled,
	max(case when order_status = 'Completed' then s_count end) as Completed,
	sum(s_count),
	round(max(case  when order_status = 'Cancelled' then s_count end)::numeric/sum(s_count)::numeric * 100,2) as percentage_cancelled,
	round(max(case when order_status = 'Completed' then s_count end)::numeric/sum(s_count)::numeric * 100, 2) as percentage_completed
from top5 as t
join order_status as os
on t.seller_id = os.seller_id
group by 1,2 
order by 1
```

### Product Profit Margin
### 12. Calculate the profit margin for each product (difference between price and cost of goods sold).
-- Challenge: Rank products by their profit margin, showing highest to lowest.
```sql
select
	product_id,
	product_name,
	profit_margin,
	dense_rank() over(order by profit_margin desc) as rn
from(select
	p.product_id,
	p.product_name,
	-- sum((p.price - p.cogs)*o.quantity) as profit,
	sum((p.price - p.cogs)*o.quantity) / sum(total_sale) * 100 as profit_margin
from order_items as o
join products as p 
on o.product_id = p.product_id
group by 1) t1
```

### Most Returned Products
### 13.Query the top 10 products by the number of returns.
-- Challenge: Display the return rate as a percentage of total units sold for each product.
```sql
select
	ot.product_id,
	p.product_name,
	count(*) as total_units_sold,
	sum(case when order_status = 'Returned' then 1 else 0 end) as no_of_returns,
	round(sum(case when order_status = 'Returned' then 1 else 0 end)::numeric/count(*)::numeric * 100, 3) as return_percentage
	
from orders as o 
join order_items as ot 
on o.order_id = ot.order_id
join products as p 
on p.product_id = ot.product_id
group by 1,2
```
### Orders Pending Shipment
### 14.Find orders that have been paid but are still pending shipment.
-- Challenge: Include order details, payment date, and customer information.
```sql
select 
	o.order_id,
	p.payment_date,
	c.customer_id,
	concat(first_name, ' ', last_name) as customer_name
from orders as  o
join order_items as ot
on o.order_id = ot.order_id
join customers as c
on o.customer_id = c.customer_id
left join payments as p
on o.order_id = p.payment_id
left join shipping as s
on o.order_id = s.order_id
where payment_status = 'Payment Successful' and delivery_status is null
```

### Inactive Sellers
### 15.Identify sellers who haven’t made any sales in the last 6 months.
-- Challenge: Show the last sale date and total sales from those sellers.
```sql
with sellers_cte as(
select
	*
from sellers
where seller_id not in (select distinct seller_id from orders where order_date >= current_date - interval '6 month')
)
select
		s.seller_id,
		max(o.order_date) as last_sale_date,
		sum(ot.total_sale) as total_sales
from orders as o
join sellers_cte as s
on o.seller_id = s.seller_id
join order_items as ot
on ot.order_id = o.order_id
group by 1
```
### IDENTITY customers into returning or new
### 16. If the customer has done more than 5 return categorize them as returning otherwise new
-- Challenge: List customers id, name, total orders, total returns

select
	customer_name,
	case when total_return > 5 then 'Returning_customers' else 'New' end as category,
	total_orders,
	total_return

from(select
	c.customer_id,
	concat(first_name, ' ', last_name) as customer_name,
	count(o.order_id) as total_orders,
	sum(case when order_status = 'Returned' then 1 else 0 end) as total_return 
from orders as o
join customers as c
on o.customer_id = c.customer_id
group by 1) t1


### Top 5 Customers by Orders in Each State
### 17. Identify the top 5 customers with the highest number of orders for each state.
-- Challenge: Include the number of orders and total sales for each customer.
```sql
with ranking_cte as(
select
	c.customer_id,
	state,
	count(o.order_id) as total_orders,
	sum(ot.total_sale) as total_sales,
	dense_rank() over(partition by c.state order by count(o.order_id) desc) as rn
from orders as o
join customers as c
on o.customer_id = c.customer_id
join order_items as ot
on ot.order_id = o.order_id
group by 1,2)
select
	*
from ranking_cte
where rn <= 5
```

### Revenue by Shipping Provider
### 18. Calculate the total revenue handled by each shipping provider.
-- Challenge: Include the total number of orders handled
```sql
select
	s.shipping_providers,
	sum(total_sale) as revenue_handled,
	count(o.order_id) as orders_handled,
from orders as o
join shipping as s
on o.order_id = s.order_id
join order_items as ot 
on ot.order_id =o.order_id
group by 1
```
### 19. Top 10 product with highest decreasing revenue ratio compare to last year(2022) and current_year(2023)
--- Challenge: Return product_id, product_name, category_name, 2022 revenue and 2023 revenue decrease ratio at end Round the result

Note: Decrease ratio = cr-ls/ls* 100 (cs = current_year ls=last_year)

```sql
with t1 as(
select
	p.product_id,
	p.product_name,
	c.category_name,
	sum(case when extract(year from order_date) = 2022 then (total_sale) end) as ls_yr,
	sum(case when extract(year from order_date) = 2023 then (total_sale) end) as cs_yr
from order_items as ot
join products as p
on ot.product_id = p.product_id
join orders as o
on o.order_id = ot.order_id
join category as c
on p.category_id = c.category_id
group by 1,2,3)
select
	product_id,
	product_name,
	category_name,
	ls_yr,
	cs_yr,
	(ls_yr - cs_yr) as revenue_difference,
	round((ls_yr - cs_yr) / ls_yr * 100,2) as revenue_ratio
from t1
where ls_yr > cs_yr
order by 7 desc
limit 10
```
### Final Task
### Stored Procedure
### create a function as soon as the product is sold the the same quantity should reduced from inventory table
#### fter adding any sales records it should update the stock in the inventory table based on the product and qty purchased
```sql
drop procedure if exists add_sales;
create or replace procedure add_sales(
	p_order_id int,
	p_customer_id int,
	p_seller_id int,
	p_order_item_id int,
	p_product_id int,
	p_quantity int
)
language plpgsql
as $$
declare
	v_count int;
	v_price numeric;
	v_product_name varchar;
begin 
	select price, product_name
	into v_price, v_product_name
	from products
	where product_id = p_product_id;
	
	select count(1)
	into v_count
	from inventory
	where product_id = p_product_id and stock >= p_quantity;

	if v_count > 0  then
		-- add into orders and order_items and update inventory table
		-- adding into orders list
		insert into orders(order_id, order_date, customer_id, seller_id)
		values
			(p_order_id, current_date, p_customer_id, p_seller_id);
		-- adding into order_items
		insert into order_items(order_item_id, order_id, product_id, quantity, price_per_unit, total_sale)
		values
			(p_order_item_id, p_order_id, p_product_id, p_quantity, v_price, v_price * p_quantity);
		-- updating inventory
		update inventory
		set stock = (stock - p_quantity)
		where product_id = p_product_id;

		raise notice 'Product: %  sold has been added to the orders and inventory is updated', v_product_name;
	else
		raise notice 'Product: % is not available', v_product_name;
	end if;
end
$$
```

**Testing Store Procedure**
```sql
call add_sales

(25005, 2, 5, 25004, 1, 14);
```

## **Learning Outcomes**

This project enabled me to:
- Design and implement a normalized database schema.
- Clean and preprocess real-world datasets for analysis.
- Use advanced SQL techniques, including window functions, subqueries, and joins.
- Conduct in-depth business analysis using SQL.
- Optimize query performance and handle large datasets efficiently.

---

## **Conclusion**

This advanced SQL project successfully demonstrates my ability to solve real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.

By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.






















