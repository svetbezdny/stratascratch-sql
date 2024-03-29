### [Most Profitable Companies](https://platform.stratascratch.com/coding/10354-most-profitable-companies?code_type=1)  

```sql
select 
  company, 
  profits
from forbes_global_2010_2014
order by profits desc
limit 3;
```

### [Workers With The Highest Salaries](https://platform.stratascratch.com/coding/10353-workers-with-the-highest-salaries?code_type=1)  

```sql
select worker_title from (
  select 
    worker_title,
    max(salary) s,
    max(max(salary)) over() as mx
  from title t, worker w
  where t.worker_ref_id = w.worker_id
  group by 1
  order by 2 desc
) t
where s = mx
```

### [Users By Average Session Time](https://platform.stratascratch.com/coding/10352-users-by-avg-session-time?code_type=1)  

```sql
with t as (
  select 
    user_id,
    to_char(timestamp, 'yyyy-mm-dd') as date,
    max(case when action = 'page_load' then timestamp else null end) as page_load,
    max(case when action = 'page_exit' then timestamp else null end) as page_exit
  from facebook_web_log
  group by 1, 2
)
select 
  user_id,
  avg(page_exit - page_load) as mean
from t
where page_exit is not null
group by 1
```

### [Finding User Purchases](https://platform.stratascratch.com/coding/10322-finding-user-purchases?code_type=1)  

```sql
select distinct user_id from (
  select 
    user_id,
    lead(created_at) over(partition by user_id order by created_at) - created_at as diff
  from amazon_transactions
  order by 1, 2
)t
where diff <= 7
```

### [Acceptance Rate By Date](https://platform.stratascratch.com/coding/10285-acceptance-rate-by-date?code_type=1)  

```sql
with s as (
  select 
    user_id_sender,
    date,
    count(*) as sent_cnt
  from fb_friend_requests
  where action = 'sent'
  group by 1,2
), a as (
  select 
    user_id_sender,
    count(*) as accepted_cnt
  from fb_friend_requests
  where action = 'accepted'
  group by 1 
)
select date, sum(accepted_cnt) / sum(sent_cnt) as percentage
from s left 
 join a using(user_id_sender)
group by 1
```

### [Ranking Most Active Guests](https://platform.stratascratch.com/coding/10159-ranking-most-active-guests?code_type=1)  

```sql
select 
  dense_rank() over(order by sum(n_messages) desc) as rnk,
  id_guest,
  sum(n_messages) as total
from airbnb_contacts
group by 2
```

### [Number Of Units Per Nationality](https://platform.stratascratch.com/coding/10156-number-of-units-per-nationality?code_type=1)  

```sql
select
  nationality,
  count( distinct unit_id) as cnt
from airbnb_hosts ah 
 join airbnb_units using(host_id)
where age < 30 and unit_type = 'Apartment'
group by 1
order by 2 desc
```

### [Find matching hosts and guests in a way that they are both of the same gender and nationality](https://platform.stratascratch.com/coding/10078-find-matching-hosts-and-guests-in-a-way-that-they-are-both-of-the-same-gender-and-nationality?code_type=1)  

```sql
select distinct
  host_id,
  guest_id
from airbnb_hosts ah join airbnb_guests ag
using(gender, nationality) ;
```

### [Income By Title and Gender](https://platform.stratascratch.com/coding/10077-income-by-title-and-gender?code_type=1)  

```sql
with t as (
  select 
    worker_ref_id as id,
    sum(bonus) as bonus
  from sf_bonus
  group by 1
)
select 
  employee_title,
  sex,
  avg(salary + bonus) as avg_compensation
from sf_employee
 join t using(id)
group by 1, 2
```

### [Highest Energy Consumption](https://platform.stratascratch.com/coding/10064-highest-energy-consumption?code_type=1)  

```sql
with common as (
  select * from fb_eu_energy
    union all
  select * from fb_asia_energy
    union all
  select * from fb_na_energy
),
t as (
  select 
    date, 
    sum(consumption) as consumption,
    max(sum(consumption)) over() as mx
  from common
  group by 1
  order by 2 desc
)
select date, consumption from t 
where consumption = mx
```

### [Top Cool Votes](https://platform.stratascratch.com/coding/10060-top-cool-votes?code_type=1)  

```sql
with t as (
  select 
    business_name,
    review_text,
    sum(cool) as cnt
  from yelp_reviews
  group by 1,2
), t2 as (
  select *, dense_rank() over(order by cnt desc) as rnk from t 
)
select 
  business_name,
  review_text,
  cnt
from t2 
where rnk = 1
```

### [Reviews of Categories](https://platform.stratascratch.com/coding/10049-reviews-of-categories?code_type=1)  

```sql
select
  unnest(string_to_array(categories, ';')),
  sum(review_count)
from yelp_business
group by 1
order by 2 desc
```

### [Top Businesses With Most Reviews](https://platform.stratascratch.com/coding/10048-top-businesses-with-most-reviews?code_type=1)  

```sql
select 
  name, 
  review_count
from yelp_business
order by 2 desc
limit 5;
```

### [Find all wineries which produce wines by possessing aromas of plum, cherry, rose, or hazelnut](https://platform.stratascratch.com/coding/10026-find-all-wineries-which-produce-wines-by-possessing-aromas-of-plum-cherry-rose-or-hazelnut?code_type=1)  

```sql
select 
  winery
from winemag_p1
where description ~* '\y(plum|cherry|rose|hazelnut)\y'
order by 1;
```

### [Top Ranked Songs](https://platform.stratascratch.com/coding/9991-top-ranked-songs?code_type=1)  

```sql
select 
  trackname,
  count(1)
from spotify_worldwide_daily_song_ranking
where position = 1
group by 1
order by 2 desc
```

### [Largest Olympics](https://platform.stratascratch.com/coding/9942-largest-olympics?code_type=1)  

```sql
select
  games,
  count(distinct id)
from olympics_athletes_events
group by 1
order by 2 desc
limit 1
```

### [Highest Cost Orders](https://platform.stratascratch.com/coding/9915-highest-cost-orders?code_type=1)  

```sql
select
  first_name,
  sum(total_order_cost),
  order_date
from customers c
 join orders o on c.id = o.cust_id
where order_date between '2019-02-01' and '2019-05-01'
group by 1, 3
order by 2 desc
limit 1;
```

### [Highest Target Under Manager](https://platform.stratascratch.com/coding/9905-highest-target-under-manager?code_type=1)  

```sql
with t as (   
  select 
    first_name,
    target,
    dense_rank() over(order by target desc) rnk
  from salesforce_employees
  where manager_id = 13
)
select first_name, target 
from t 
where rnk = 1
```

### [Highest Salary In Department](https://platform.stratascratch.com/coding/9897-highest-salary-in-department?code_type=1)  

```sql
select department, first_name, salary from (
  select 
    department,
    first_name,
    salary,
    dense_rank() over (partition by department order by salary desc) rnk
  from employee
) t
where rnk=1
```

### [Employee and Manager Salaries](https://platform.stratascratch.com/coding/9894-employee-and-manager-salaries?code_type=1)  

```sql
select first_name, e_sale from (
  select 
    e.first_name,
    sum(e.salary) as e_sale,
    sum(m.salary) as m_sale
  from employee e, employee m
  where e.manager_id = m.id
  group by 1
) t
where e_sale > m_sale
```

### [Customer Revenue In March](https://platform.stratascratch.com/coding/9782-customer-revenue-in-march?code_type=1)  

```sql
select * from (
  select 
    cust_id,
    sum(total_order_cost) filter (where extract(month from order_date) = 3) as revenue
from orders
group by 1
) t
where revenue is not null
order by 2 desc
```

### [Find the rate of processed tickets for each type](https://platform.stratascratch.com/coding/9781-find-the-rate-of-processed-tickets-for-each-type?code_type=1)  

```sql
select 
  type,
  1.0 * count(*) filter (where processed = True) / count(*) as processed_rate
from facebook_complaints
group by 1
```

### [Number of violations](https://platform.stratascratch.com/coding/9728-inspections-that-resulted-in-violations?code_type=1)  

```sql
select 
  extract(year from inspection_date) as year,
  count(*) as cnt
from sf_restaurant_health_violations
where violation_id is not null and business_name = 'Roxanne Cafe'
group by 1
order by 1;
```

### [Classify Business Type](https://platform.stratascratch.com/coding/9726-classify-business-type?code_type=1)  

```sql
select distinct
  business_name,
  case when business_name ilike '%restaurant%' then 'restaurant'
   when business_name ilike '%caf%' or business_name ilike '%cof%' then 'cafe'
   when business_name ilike '%school%' then 'school'
   else 'other'
   end as classification
from sf_restaurant_health_violations;
```

### [Find the top 10 ranked songs in 2010](https://platform.stratascratch.com/coding/9650-find-the-top-10-ranked-songs-in-2010?code_type=1)  

```sql
select distinct
  year_rank,
  group_name,
  song_name
from billboard_top_100_year_end
where year = 2010
order by 1
limit 10;
```

### [Activity Rank](https://platform.stratascratch.com/coding/10351-activity-rank?code_type=1)  

```sql
select 
  from_user,
  count(*) as total_emails,
  row_number() over(order by count(*) desc, from_user) as row_number
from google_gmail_emails
group by from_user
```

### [New Products](https://platform.stratascratch.com/coding/10318-new-products?code_type=1)  

```sql
select 
  company_name,
  sum(case when year = 2020 then 1 else 0 end) - 
   sum(case when year = 2019 then 1 else 0 end) as net_products
from car_launches
group by 1
```

### [Risky Projects](https://platform.stratascratch.com/coding/10304-risky-projects?code_type=1)  

```sql
select
 title,
 budget,
 ceil(sum(salary) * sum(end_date - start_date))
from linkedin_projects lp join (
  select project_id, sum(salary) / 365 as salary
  from linkedin_emp_projects lep
  join linkedin_employees le on lep.emp_id = le.id
  group by 1
 ) t on lp.id = t.project_id
group by 1, 2
having ceil(sum(salary) * sum(end_date - start_date)) > budget
order by 1
```

### [Second Highest Salary](https://platform.stratascratch.com/coding/9892-second-highest-salary?code_type=1)  

```sql
select distinct salary from employee
order by salary desc
limit 1 offset 1
```

### [Find the percentage of shipable orders](https://platform.stratascratch.com/coding/10090-find-the-percentage-of-shipable-orders?code_type=1)  

```sql
select 
 100.0 * count(*) filter (where address is not null) / count(*) as percent_shipable
from orders
join customers on orders.cust_id = customers.id
```

### [Premium Acounts](https://platform.stratascratch.com/coding/2097-premium-acounts?code_type=1)  

```sql
select 
 a.entry_date,
 count(a.account_id) as premium_paid_accounts,
 count(b.account_id) as premium_paid_accounts_after_7d
from premium_accounts_by_day a
left join premium_accounts_by_day b on a.account_id = b.account_id
 and b.entry_date - a.entry_date = 7
 and b.final_price != 0
where a.final_price != 0
group by 1
order by 1
limit 7
```

### [Find students with a median writing score](https://platform.stratascratch.com/coding/9610-find-students-with-a-median-writing-score?code_type=1)  

```sql
with t (median) as (
 select percentile_disc(0.5) within group (order by sat_writing) from sat_scores
)
select student_id 
from sat_scores, t
where sat_writing = median
```

### [User with Most Approved Flags](https://platform.stratascratch.com/coding/2104-user-with-most-approved-flags?code_type=1)  

```sql
select username from (
  select 
   concat_ws(' ', user_firstname, user_lastname) as username,
   dense_rank() over (order by count(distinct video_id) desc) as rn
  from user_flags a
  join (
    select flag_id from flag_review
    where reviewed_outcome = 'APPROVED'
  ) b using(flag_id)
  group by 1
)t where rn = 1
```

### [Find the number of times each word appears in drafts](https://platform.stratascratch.com/coding/9817-find-the-number-of-times-each-word-appears-in-drafts?code_type=1)  

```sql
select 
 unnest(string_to_array(replace(trim(contents, '.'), ',', ''), ' ')) as word,
 count(*) as nentry
from google_file_store
where filename like 'draft%'
group by 1
```

### [Election Results](https://platform.stratascratch.com/coding/2099-election-results?code_type=1)  

```sql
select candidate from (
 select 
  candidate, 
  dense_rank() over(order by  sum(g) desc) as rn
from (
  select
   candidate,
   1.0 * 1 / count(*) over (partition by voter) as g
  from voting_results
  where candidate is not null
 )t
 group by 1
) foo where rn = 1
```

### [Apple Product Counts](https://platform.stratascratch.com/coding/10141-apple-product-counts?code_type=1)  

```sql
select 
 language,
 count(distinct user_id) filter (where device in ('macbook pro', 'iphone 5s', 'ipad air')) as n_apple_users,
 count(distinct user_id) as n_total_users
from playbook_events
join playbook_users using(user_id)
group by 1
order by 3 desc
```

### [Make a report showing the number of survivors and non-survivors by passenger class](https://platform.stratascratch.com/coding/9881-make-a-report-showing-the-number-of-survivors-and-non-survivors-by-passenger-class?code_type=1)  

```sql
select 
 survived,
 sum(case when pclass = 1 then 1 else 0 end) as first_class,
 sum(case when pclass = 2 then 1 else 0 end) as second_class,
 sum(case when pclass = 3 then 1 else 0 end) as third_class
from titanic
group by 1
```

### [Flags per Video](https://platform.stratascratch.com/coding/2102-flags-per-video?code_type=1)  

```sql
select 
 video_id,
 count(distinct concat(user_firstname, user_lastname)) as num_unique_users
from user_flags
where flag_id is not null
group by 1
```

### [Share of Active Users](https://platform.stratascratch.com/coding/2005-share-of-active-users?code_type=1)  

```sql
select 
 count(*) filter (where status = 'open')::decimal / count(*) as active_users_share
from fb_active_users
where country = 'USA'
```

### [Spam Posts](https://platform.stratascratch.com/coding/10134-spam-posts?code_type=1)  

```sql
select 
 post_date,
 100.0 * count(*) filter (where post_keywords like '%spam%') / count(*) as spam_share
from facebook_posts
join facebook_post_views using(post_id)
group by 1
```