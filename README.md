# Coffee-Sales-Analysis---SQL-PowerBI
Analyzed 15 business problems for a Coffee shop using SQL, extracting data to evaluate business performance and growth. Utilized advanced SQL queries like split_part, aggregations, and subqueries to derive insights and support business growth strategies. Same data is then shown in the form of charts, graphs and heatmaps with the help of PowerBI.

## Process Flow :
First of all after studying the given dataset of Coffee shop, necessary data cleaning has been made and then 15 different business related queries were fired through MySQL, then used same table in PowerBI to generate insights in the form of visuals (Horizontal & Vertical bar graphs, Calender chart, Months filter, Number cards and Line graphs).

## MySQL Part :
### -- DATA CLEANING
SELECT * from coffee_shop_sales;

SET SQL_SAFE_UPDATES = 0;

update coffee_shop_sales
set transaction_date = str_to_date(transaction_date, '%d-%m-%Y');

alter table coffee_shop_sales
modify column transaction_date date;

update coffee_shop_sales
set transaction_time = str_to_date(transaction_date, '%H:%i:%s');

alter table coffee_shop_sales
modify column transaction_time time;
describe coffee_shop_sales;

alter table coffee_shop_sales
change column transaction_id transaction_id int;

### Main Queries :
1 MONTH WISE TOTAL SALES :
``` sql
select * from coffee_shop_sales;

select date_format(transaction_date,'%M') as month_name, round(sum(unit_price * transaction_qty),0) as sales
from coffee_shop_sales
group by month_name;
```
2 Month on Month increase or decrease in sales in current month wrt previous month
``` sql
SELECT 
MONTH(transaction_date) AS month,
ROUND(SUM(unit_price * transaction_qty)) AS total_sales,

    (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty), 1)
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(unit_price * transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
coffee_shop_sales
group by MONTH(transaction_date);
```
3 Total no of orders for each respective months
``` sql
select * from coffee_shop_sales;

select date_format(transaction_date,'%M') as month_name, count(transaction_id) as total_orders
from coffee_shop_sales
group by month_name;
```
4 MoM increase or decrease in orders in current month wrt previous month
``` sql
select date_format(transaction_date,'%M') as month_name, count(transaction_id) as total_orders
(count(transaction_id) - lag(count(transaction_id),1) over (order by month_name)) / lag(count(transaction_id),1) over (order by month_name) * 100 as per_inc_dec  
from coffee_shop_sales
group by month_name;
```

5 Total qty sold for each respective month
``` sql
select * from coffee_shop_sales;

select date_format(transaction_date, '%M') as month_name, sum(transaction_qty) as total_qty
from coffee_shop_sales
group by month_name;
```

6 MoM increase or decrease in total qty sold in current month wrt previous month
``` sql
SELECT 
MONTH(transaction_date) AS month,
    ROUND(SUM(transaction_qty)) AS total_quantity_sold,
    (SUM(transaction_qty) - LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(transaction_qty), 1) 
    OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM 
coffee_shop_sales
group by MONTH(transaction_date);
```

7 CALENDAR TABLE – DAILY SALES, QUANTITY and TOTAL ORDERS
``` sql
SELECT
CONCAT(ROUND(SUM(unit_price * transaction_qty)/1000, 1),"K") AS total_sales,
SUM(transaction_qty) AS total_quantity_sold,
COUNT(transaction_id) AS total_orders
FROM    
coffee_shop_sales
where transaction_date = "2023-03-27";
```
8 sales of weekends and on weekdays
``` sql
select * from coffee_shop_sales;

select date_format(transaction_date, "%M") as month_name,
case when dayofweek(transaction_date) in (1,7) then "weekends"
else "weekdays"
end as day_type,
sum(unit_price * transaction_qty) as total_sales
from coffee_shop_sales
group by date_format(transaction_date, "%M"), case when dayofweek(transaction_date) in (1,7) then "weekends"
else "weekdays"
end;
```

9 sales data by store location
``` sql
select * from coffee_shop_sales;

select date_format(transaction_date, "%M") as month_name, store_location, concat(round(sum(unit_price * transaction_qty)/1000,1), "K") as total_sales,
(concat(round(sum(unit_price * transaction_qty)/1000,1), "K") - LAG(concat(round(sum(unit_price * transaction_qty)/1000,1), "K"),3) 
OVER (ORDER BY date_format(transaction_date, "%M"))) / LAG(concat(round(sum(unit_price * transaction_qty)/1000,1), "K"),3) 
OVER (ORDER BY date_format(transaction_date, "%M")) * 100 as mom_inc_dec
from coffee_shop_sales
group by date_format(transaction_date, "%M"), store_location;
```

10 Daily sales analysis
``` sql
select * from coffee_shop_sales;

-- MONTH WISE AVERAGE SALES
select avg(total_sales) from
(select transaction_date, concat(round(sum(unit_price * transaction_qty)/1000,1), "K") as total_sales
from coffee_shop_sales
where month(transaction_date) = 5
group by transaction_date) as a;
```
-- MONTH WISE DAILY SALES
``` sql
select transaction_date, concat(round(sum(unit_price * transaction_qty)/1000,1), "K") as total_sales
from coffee_shop_sales
where month(transaction_date) = 5
group by transaction_date;
```
-- COMPARING DAILY SALES WITH AVERAGE SALES – IF GREATER THAN “ABOVE AVERAGE” and LESSER THAN “BELOW AVERAGE”
``` sql
SELECT DAY(transaction_date) AS day_of_month,
SUM(unit_price * transaction_qty) AS total_sales,
AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5  -- Filter for May
GROUP BY DAY(transaction_date)
ORDER BY day_of_month asc;
```

``` sql
select day_of_month, total_sales, avg_sales,
case when total_sales > avg_sales then "above_avg"
when total_Sales < avg_sales then "below_avg"
else "average"
end as conclusion
from (SELECT DAY(transaction_date) AS day_of_month,
SUM(unit_price * transaction_qty) AS total_sales,
AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5  -- Filter for May
GROUP BY DAY(transaction_date)
ORDER BY day_of_month asc) as b;
```
11 sales by product catagory
``` sql
select * from coffee_shop_sales;

select month(transaction_date) as month_no, product_category,sum(unit_price * transaction_qty) as total_sales
from coffee_shop_sales
group by month(transaction_date), product_category;
```
12 top 10 products by sales
``` sql
SELECT 
	product_type,
	ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE
	MONTH(transaction_date) = 5 and product_category = "Coffee"
GROUP BY product_type
ORDER BY SUM(unit_price * transaction_qty) DESC
LIMIT 10;
```

13 SALES BY DAY | HOUR
``` sql
SELECT 
ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales,
SUM(transaction_qty) AS Total_Quantity,
COUNT(*) AS Total_Orders
FROM 
coffee_shop_sales
WHERE
DAYOFWEEK(transaction_date) = 3 -- Filter for Tuesday (1 is Sunday, 2 is Monday, ..., 7 is Saturday)
    AND HOUR(transaction_time) = 8 -- Filter for hour number 8
    AND MONTH(transaction_date) = 5; -- Filter for May (month number 5)
```

14 HOUR WISE TOTAL SALES
``` sql
select hour(transaction_time) as hours, concat(round(sum(unit_price * transaction_qty)/1000, 1), "K") as total_Sales
from coffee_shop_sales
where month(transaction_date) = 5
group by hour(transaction_time)
order by hour(transaction_time);
```

15 DAY WISE TOTAL SALES
``` sql
select dayname(transaction_date) as days, concat(round(sum(unit_price * transaction_qty)/1000, 1), "K") as total_Sales
from coffee_shop_sales
where month(transaction_date) = 5
group by dayname(transaction_date);
order by dayname(transaction_date);
```

## PowerBI Part :
Generated interactive dashboards showing visuals in the form of charts, graphs and heatmaps for the same above problems. We already derived required business driven insights firing SQL queries but to create visuals with interactive dashboard which shows month wise filter datas of daily and monthlu values of total sales, total orders and total quantities, PowerBI is used. 
Apart from this visuals like Donut charts for sales data for weekends and weekdays, bargraphs showing daily sales, top products and product catagories in the terms of sales and hour & day name wise sales data.
Also create a tool tip for total sales, total orders and total quantities which can be shown while hovering cursor on any particular date of calender chart just for detail study of that particular day. 

## Screenshot of Dashboard :
![Dashboard](https://github.com/Bonny1309/Coffee-Sales-Analysis---SQL-PowerBI/blob/main/Dashboard.PNG)















