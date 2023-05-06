# Foody_fi
Case study 3_by Danny Ma

A. Customer Journey
```
select customer_id, plan_name,SUBSTRING(CAST(start_date AS text) FROM 1 FOR 10) AS start_date_date
from foodie_fi.subscriptions s
join foodie_fi.plans p
on s.plan_id = p.plan_id
where customer_id = 1;
```

B. Data Analysis Questions

```
Select Count(distinct customer_id) from foodie_fi.subscriptions;

select extract(month from start_date) as month, count(customer_id) 
from foodie_fi.subscriptions
where plan_id = 0
group by 1
order by 2 desc;

select s.plan_id,p.plan_name, count(*)
from foodie_fi.subscriptions s
join foodie_fi.plans p
on s.plan_id = p.plan_id
where start_date >= '2021=01-01'
group by s.plan_id,p.plan_name;

with cte as (
select count(distinct customer_id) as total_cust,
count(case when plan_id= 4 then 1 else null end) as churn_cnt
from foodie_fi.subscriptions)
select total_cust as total_customer, (cast (churn_cnt as float)/cast (total_cust as float)) * 100.0 as churn_percent
from cte;

with cte as
(select *, 
lag(plan_id,1) over(partition by customer_id order by plan_id) as prev
from foodie_fi.subscriptions)
select count(customer_id) as total_churn_num,
Round(((count(customer_id)*1.0)/(select (count(distinct customer_id)*1.0) from foodie_fi.subscriptions)*100),0) as churn_percent
from cte 
where prev = 0 and plan_id = 4;

with cte as
(select customer_id,plan_id, lead(plan_id,1) over(partition by customer_id order by plan_id) as post from foodie_fi.subscriptions)

select count(customer_id), post,
Round(((count(customer_id)*1.0)/(select (count(distinct customer_id)*1.0) from foodie_fi.subscriptions)*100),2) as cust_percent
from cte 
where post is not null and plan_id = 0
group by post;

with cte as 
(select count(customer_id) as cust_cnt, p.plan_name
from foodie_fi.subscriptions s
join foodie_fi.plans p
on s.plan_id = p.plan_id
where start_date <= '2020-12-31'
group by p.plan_name)
select plan_name, cust_cnt as customer_count, Round ((cust_cnt * 1.0/(select count(*) from foodie_fi.subscriptions where start_date <= '2020-12-31'))*100.0,2) as percentage
from cte;

with cte as 
(select customer_id, s.plan_id, plan_name,
lead(s.plan_id,1) over(partition by customer_id order by s.plan_id) as next_plan
from foodie_fi.subscriptions s
join foodie_fi.plans p
on s.plan_id = p.plan_id
where start_date between '2020-01-01' and '2020-12-31')
select count(distinct customer_id)
from cte 
where plan_id < 3 and next_plan = 3;

with joiners as 
(select customer_id, start_date as join_date 
from foodie_fi.subscriptions
where plan_id = 0),
Upgrade_to_annual as
(select customer_id, start_date as annual_sub
 from foodie_fi.subscriptions
where plan_id = 3)
 select Round(AVG(annual_sub - join_date),0) as avg_days
 from joiners j
 join Upgrade_to_annual u
 on u.customer_id = j.customer_id;
 
 with joiners as 
(select customer_id, start_date as join_date 
from foodie_fi.subscriptions
where plan_id = 0),
Upgrade_to_annual as
(select customer_id, start_date as annual_sub
 from foodie_fi.subscriptions
where plan_id = 3)
 select Round(AVG(annual_sub - join_date),0) as avg_days, extract (day from start_date) as day
 from joiners j
 join Upgrade_to_annual u
 on u.customer_id = j.customer_id
 join foodie_fi.subscriptions s
 on s.customer_id = u.customer_id
 group by 2
 order by 2;
 
 with cte as
(select customer_id, plan_name,s.plan_id,start_date,
lead(s.plan_id,1) over(partition by customer_id order by s.plan_id) as next_plan
from foodie_fi.subscriptions s
join foodie_fi.plans p
on s.plan_id = p.plan_id)
select count(customer_id) as Customer_downgraded
from cte 
where plan_id = 2 and next_plan = 1
and start_date between '2020-01-01' and '2020-12-31';
```
