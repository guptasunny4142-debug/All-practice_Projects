# All-Projects

-- Apple Sales Project with 1M rows seles Datasets

```select * from category;
select * from products;
select * from sales;
select * from stores;
select * from warranty;
```
-- EDA
```select distinct repair_status from warranty;
select count(*) from sales;
```
-- **Improving Query Performance**

-- ET - 126.ms

-- PT - 0.157 ms

-- ET After index - 30-40 ms

```EXPLAIN ANALYZE

select * from sales
where product_id = '3'

create index sales_product_id on sales(product_id);
```

-- ET - 121.ms

-- PT - 0.153ms

-- ET After index - 40-50ms
```
EXPLAIN ANALYZE
select * from sales
where store_id = '10'

create index sales_store_id on sales(store_id);
```
-- **Business Problem**

-- 1.Find the number of stores in each country.
```
select  country,
        count(store_id) as tatal_store
from stores
Group by 1
order by 2 desc
```
-- 2.Calculate the total number of units sold by each store.
```
select s.store_id,
       st.store_name,
       sum(s.quantity) as Total_unit_sales
from sales as s
join stores as st
on st.store_id = s.store_id
group by 1, 2
order by 3 desc
```
-- 3.Identify how many sales occurred in December 2023.
```
select count(sale_id) as total_sales
FROM sales
where TO_CHAR(sale_date, 'MM-YYYY') = '12-2023'
```
-- 4.Determine how many stores have never had a warranty claim filed.
```
SELECT count(*) as stores_without_warranty_claim
FROM stores
WHERE store_id not in (
SELECT
DISTINCT store_id
FROM sales as s
RIGHT JOIN warranty as w
ON s.sale_id = w.sale_id
);
```
-- 5. Calculate the percentage of warranty claims marked as "Warranty Void".
```
select  
     round
	 (count(claim_id) / 
	  (select count(*) from warranty) :: numeric
       * 100, 2)
from warranty 
where repair_status = 'Warranty Void'
```
-- 6. Identify which store had the highest total units sold in the last year.
```
select s.store_id,
       st.store_name,
       sum(s.quantity) as Total_unit_sold
from sales as s
join stores as st
on s.store_id = st.store_id
where sale_date >= ( current_date - interval '2 year')
group by 1 , 2
order by 3 desc
Limit 1
```
-- 7. Count the number of unique products sold in the last year.
```
select 
      count(distinct product_id)
from sales
where sale_date >= (current_date - interval '2 year')
```
-- 8. Find the average price of products in each category.
```
select 
      p.category_id,
	  c.category_name,
	  ROUND(avg(price),2) as avg_price_product	  
from products as p
join category as c
on p.category_id = c.category_id
group by 1, 2
order by 3 desc
LIMIT 10
```
-- 9. How many warranty claims were filed in 2023?
```
select
     count(*) as warranty_claim
from warranty 
where extract (year from  claim_date ) = '2023'
```
-- 10. For each store, identify the best-selling day based on highest quantity sold.
--  STORE_ID , DAY_NAME, SUM(QTY)
-- WINDOW FUNCTION
```
select * 
from 
(select s.store_id,
       st.store_name,
      to_char(s.sale_date, 'DAY') as DAY_NAME,
	  SUM(s.quantity) as total_unit_sold,
	  RANK() OVER(partition by s.store_id order by sum(s.quantity) desc ) as rank
from sales as s
join stores as st
on s.store_id = st.store_id
group by 1 , 2, 3
)
as t1
where rank = 1
```
-- **Medium to Hard Questions**

-- 11. Identify the least selling product in each country for each year based on total units sold.
```
with product_rank
as 
(
select st.country,
       p.product_name,
	   sum(s.quantity) as total_quntity_sold,
	   rank() over(partition by st.country order by sum(s.quantity))as rank 
from sales as s 
join
stores as st 
on s.store_id = st.store_id
join 
products as p
on s.product_id = p.product_id
group by 1, 2
)
select * 
from product_rank
where rank = 1
```

-- 12. Calculate how many warranty claims were filed within 180 days of a product sale.
```
select count(*)
 from warranty as w
left join 
sales as s 
on s.sale_id = w.sale_id 
where w.claim_date >= s.sale_date
      and
     w.claim_date - s.sale_date <= 180
```	 
-- 13. Determine how many warranty claims were filed for products launched in the last two years.
```
select p.product_name,
       count(w.claim_id) as number_claim,
	   count(s.sale_id) as total_sales
from warranty as w
right join 
sales as s 
on s.sale_id = w.sale_id
join 
products as p
on p.product_id = s.product_id
where launch_date >= current_date - interval '3 year'
group by 1
order by 2 , 3 desc
```
-- 14. List the months in the last three years where sales exceeded 5,000 units in the USA.
```
select 
      to_char(sale_date, 'MM-YYY') AS Month,
	  sum(s.quantity) as total_unit_sold	  
from sales as s
join 
stores as st 
on s.store_id = st.store_id
where country = 'USA'
      and 
	  s.sale_date >= current_date - interval '3 year'
group by 1
having sum(quantity) > 5000
```
-- 15. Identify the product category with the most warranty claims filed in the last two years.
```
select c.category_name,
       count(w.claim_id) as total_claim
from 
warranty as w
join 
sales as s
on w.sale_id = s.sale_id
join 
products as p
on p.product_id = s.product_id
join 
category as c 
on c.category_id = p.category_id
where claim_date >= current_date - interval '2 year'
group by 1
order by 2 desc
```
-- **Complex Problems**

-- 16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```
select country,
       total_unit_sold,
	   total_claim,
	   ROUND(total_claim :: numeric / total_unit_sold :: numeric * 100, 2) AS
	   Rick
	   
from	   
(select
       st.country,
       sum(s.quantity) as total_unit_sold,
	   count(w.claim_id) as total_claim 
from sales as s
join 
stores  as st
on s.store_id = st.store_id
left join
warranty as w
on w.sale_id = s.sale_id
group by 1)
t1
order by 4 desc
```
-- 17. Analyze the year-by-year growth ratio for each store.
```
with yearly_sales
as
(select 
       s.store_id,
       st.store_name,
       extract (year from sale_date) as year,
       sum(s.quantity * p.price) as total_sale       
from sales as s
join products as p 
  on s.product_id = p.product_id
join stores as st 
  on st.store_id = s.store_id
group by 1, 2, 3
order by 1, 3 
),
 growth_ratio
as
(
  select
        store_name,
         year,
	     lag(total_sale, 1) over(partition by store_name order by year) as last_year_sale,
	     total_sale as current_year_sale
from yearly_sales
) 
select store_name,
         year,
		 last_year_sale,
		 current_year_sale,
		 ROUND((current_year_sale - last_year_sale) :: numeric / last_year_sale :: numeric * 100, 2)
from growth_ratio
where last_year_sale is not null
      and 
      year <> extract(year from current_date - interval '0 year')
```	  
-- 18. Calculate the correlation between product price and warranty claims for products sold in the last five years,
-- segmented by price range.
```
select 
        case when p.price < 500 then  'less expenses product'
	    when p.price between 500 and 1000 then 'Mid Range Product'
	    else 'expensive product'
	    end as price_segment,
        count(w.claim_id) as total_claim   
from warranty as w 
left join sales as s 
on w.sale_id = s.sale_id
join products as p
on p.product_id = s.product_id
where w.claim_date >= current_date - interval '5 year'
group by 1
order by 2 desc
```
-------------------------------------------------------------------------------------------------------------------------------

select 
      case when p.price < 500 then 'less expence product'
	  when p.price between 500 and 1000 then 'Mid range product'
	  else 'expensive Product'
	  end as price_segment,
	  count(w.claim_id) as total_claim
from warranty as w
join sales as s 
on w.sale_id = s.sale_id
join products as p
on p.product_id = s.product_id
where claim_date >=current_date - interval '5 year'
group by 1
order by 2
	  
-- 19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.  
```
with Paid_Repaired
as
(select 
       s.store_id,
	   count(w.claim_id) as Paid_Repaired
from warranty as w
Right join sales as s 
on w.sale_id = s.sale_id
where repair_status = 'Paid Repaired'
group by 1
),

total_Repaired 
as
(select 
       s.store_id,
	   count(w.claim_id) as total_Repaired
from warranty as w
right join sales as s 
on w.sale_id = s.sale_id
group by 1
)
select 
      tp.store_id,
	  st.store_name,
	  pr.Paid_Repaired,
	  tp.total_Repaired,
	  ROUND( pr.Paid_Repaired::numeric/ tp.total_Repaired ::  numeric *100, 2 )as percentage_paid_repaired
from Paid_Repaired as pr
join total_Repaired  tp
on pr.store_id = tp.store_id
join stores as st 
on tp.store_id = st.store_id
```
--------------------------------------------------------------------------------------------------------------------------
select * from warranty
with paid_Repaired
as
(
  select 
       s.store_id,
	   count(w.repair_status) as paid_Repaired
from warranty as w 
Right Join sales as s 
on w.sale_id = s.sale_id
where repair_status = 'Paid Repaired'
group by 1
),

Other_repaired
as 
(
  select 
         s.store_id,
		 count(w.repair_status) as Other_repaired
  from warranty as w 
  Right Join sales as s 
  on w.sale_id = s.sale_id
  group by 1  
)
select 
      pr.store_id,
	  st.store_name,
	  pr.paid_Repaired,
	  tr.Other_repaired,
	  ROUND(pr.paid_Repaired ::numeric/tr.Other_repaired ::numeric *100, 2) AS Percentage_paid_repaired
from paid_Repaired as pr
join Other_repaired as tr
on pr.store_id = tr.store_id
join stores as st
on st.store_id =pr.store_id

-- 20. Write a query to calculate the monthly running total of sales for each store over the past four years
-- and compare trends during this period.	  
	  
WITH monthly_sale
AS
(
select 
      s.store_id,
	  extract(year from sale_date) as year,
	  extract(month from sale_date) as month,
	  sum(p.price * s.quantity) as Total_revenue
from sales as s 
join products as p 
on p.product_id = s.product_id
group by 1, 2, 3
order by 1, 2, 3
)
SELECT 
       store_id,
	   month,
	   year,
	   Total_revenue,
	   sum(Total_revenue) over(partition by store_id order by month, year ) as running_total
from monthly_sale	   

-- 12. Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months,
-- 12-18 months, and beyond 18 months.
	333
	3
select 
     p.product_name,
	   case
           when s.sale_date between p.launch_date and p.launch_date + interval '6 month' then '0-6 month'
		   when s.sale_date between p.launch_date + interval '6 month' and p.launch_date + interval '12 month' then '6-12 month'
		   when s.sale_date between p.launch_date + interval '12 month' and p.launch_date + interval '18 month' then '12-18 month'
		   else '18 month'
		   end as launch_periods,
	 sum(s.quantity) as Total_sale 	   
from sales as s
join products as p 
on s.product_id = p.product_id
group by 1 , 2
order by 1 , 3 desc
