### [Number Of Bathrooms And Bedrooms](https://platform.stratascratch.com/coding/9622-number-of-bathrooms-and-bedrooms?code_type=1)  

```sql
select 
  city,
  property_type,
  avg(bathrooms) as bathrooms_mean,
  avg(bedrooms) as bedrooms_mean
from airbnb_search_details
group by 1, 2
```

### [Count the number of user events performed by MacBookPro users](https://platform.stratascratch.com/coding/9653-count-the-number-of-user-events-performed-by-macbookpro-users?code_type=1)  

```sql
select 
  event_name,
  count(*)
from playbook_events
where device ~* 'macbook pro'
group by 1
order by 2 desc;
```

### [Churro Activity Date](https://platform.stratascratch.com/coding/9688-churro-activity-date?code_type=1)  

```sql
select 
  activity_date,
  pe_description
from los_angeles_restaurant_health_inspections
where facility_name = 'STREET CHURROS' and score < 95
```

### [Customer Details](https://platform.stratascratch.com/coding/9891-customer-details?code_type=1)  

```sql
select 
  first_name, 
  last_name,
  city,
  order_details
from customers c left join orders o on c.id = o.cust_id
order by 1, 4
```

### [Order Details](https://platform.stratascratch.com/coding/9913-order-details?code_type=1)  

```sql
select 
  first_name,
  order_date,
  order_details,
  total_order_cost
from customers c
  left join orders o  on c.id = o.cust_id
where first_name in ('Jill', 'Eva')
order by cust_id;
```

### [Average Salaries](https://platform.stratascratch.com/coding/9917-average-salaries?code_type=1)  

```sql
select 
  department,
  first_name,
  salary,
  avg(salary) over(partition by department) as avg_salary
from employee;
```

### [Find libraries who haven't provided the email address in circulation year 2016 but their notice preference definition is set to email](https://platform.stratascratch.com/coding/9924-find-libraries-who-havent-provided-the-email-address-in-2016-but-their-notice-preference-definition-is-set-to-email?code_type=1)  

```sql
select
  distinct home_library_code
from library_usage
where provided_email_address = False and notice_preference_definition = 'email';
```

### [Find the base pay for Police Captains](https://platform.stratascratch.com/coding/9972-find-the-base-pay-for-police-captains?code_type=1)  

```sql
select 
  employeename,
  basepay
from sf_public_salaries
where  jobtitle ilike '%police%'
```

### [Find how many times each artist appeared on the Spotify ranking list](https://platform.stratascratch.com/coding/9992-find-artists-that-have-been-on-spotify-the-most-number-of-times?code_type=1)  

```sql
select 
  artist,
  count(*)
from spotify_worldwide_daily_song_ranking
group by 1
order by 2 desc;
```

### [Lyft Driver Wages](https://platform.stratascratch.com/coding/10003-lyft-driver-wages?code_type=1)  

```sql
select * from lyft_drivers
where (yearly_salary <= 30000) or (yearly_salary > 70000)
```

### [Popularity of Hack](https://platform.stratascratch.com/coding/10061-popularity-of-hack?code_type=1)  

```sql
select 
  location,
  avg(popularity) as mean
from facebook_employees fe 
join facebook_hack_survey fhs on fe.id = fhs.employee_id
group by 1
```

### [Find all posts which were reacted to with a heart](https://platform.stratascratch.com/coding/10087-find-all-posts-which-were-reacted-to-with-a-heart?code_type=1)  

```sql
select * from facebook_posts
where post_id in (select distinct post_id from facebook_reactions where reaction = 'heart')
```

### [Count the number of movies that Abigail Breslin nominated for oscar](https://platform.stratascratch.com/coding/10128-count-the-number-of-movies-that-abigail-breslin-nominated-for-oscar?code_type=1)  

```sql
select 
  count(1)
from oscar_nominees
where nominee ~* 'Breslin';
```

### [Bikes Last Used](https://platform.stratascratch.com/coding/10176-bikes-last-used?code_type=1)  

```sql
select 
  bike_number,
  max(end_time) as last_used
from dc_bikeshare_q1_2012
group by 1
order by 2 desc
```

### [Finding Updated Records](https://platform.stratascratch.com/coding/10299-finding-updated-records?code_type=1)  

```sql
select 
  id,
  first_name,
  last_name,
  department_id,
  max(salary) as current_salary
from ms_employee_salary
group by 1, 2, 3, 4
order by id
```

### [Salaries Differences](https://platform.stratascratch.com/coding/10308-salaries-differences?code_type=1)  

```sql
with marketing as (
  select max(salary) as s1 from db_employee
  where department_id = (select id from db_dept where department = 'marketing')
), engineering as( 
  select max(salary) as s2 from db_employee
  where department_id = (select id from db_dept where department = 'engineering')
)
select 
  s1 - s2 as salary_difference
from marketing, engineering
```

### [Reviews of Hotel Arena](https://platform.stratascratch.com/coding/10166-reviews-of-hotel-arena?code_type=1)  

```sql
select 
 hotel_name,
 reviewer_score,
 count(*)
from hotel_reviews
where hotel_name = 'Hotel Arena'
group by 1, 2
```

### [Admin Department Employees Beginning in April or Later](https://platform.stratascratch.com/coding/9845-find-the-number-of-employees-working-in-the-admin-department?code_type=1)  

```sql
select count(*) as n_admins from worker
where department = 'Admin'
 and extract('month' from joining_date) >= 4
```

### [Number of Workers by Department Starting in April or Later](https://platform.stratascratch.com/coding/9847-find-the-number-of-workers-by-department?code_type=1)  

```sql
select 
 department,
 count(*) as num_workers
from worker
where extract('month' from joining_date) >= 4
group by 1
order by 2 desc
```

### [Number of Shipments Per Month](https://platform.stratascratch.com/coding/2056-number-of-shipments-per-month?code_type=1)  

```sql
select 
 to_char(shipment_date, 'yyyy-mm') as year_month,
 count(concat(shipment_id, sub_id))
from amazon_shipment
group by 1
```

### [Most Lucrative Products](https://platform.stratascratch.com/coding/2119-most-lucrative-products?code_type=1)  

```sql
select
 product_id,
 sum(units_sold * cost_in_dollars) as units_sold
from online_orders
group by 1
order by 2 desc
limit 5
```

### [Unique Users Per Client Per Month](https://platform.stratascratch.com/coding/2024-unique-users-per-client-per-month?code_type=1)  

```sql
select
 client_id,
 extract('month' from time_id) as month,
 count(distinct user_id) as users_num
from fact_events
group by 1, 2
```