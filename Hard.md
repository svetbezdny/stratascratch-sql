### [Monthly Percentage Difference](https://platform.stratascratch.com/coding/10319-monthly-percentage-difference?code_type=1)  

```sql
select 
  to_char(created_at, 'yyyy-mm') as y_m,
  round( 100.0 * (sum(value) - 
    lag(sum(value)) over( order by to_char(created_at, 'yyyy-mm')) ) /
      lag(sum(value)) over( order by to_char(created_at, 'yyyy-mm')), 2) as diff
from sf_transactions
group by 1
order by 1
```

### [Premium vs Freemium](https://platform.stratascratch.com/coding/10300-premium-vs-freemium?code_type=1)  

```sql
select * from (
  select 
    date,
    sum(case when paying_customer = 'no' then downloads else 0 end) as non_paying,
    sum(case when paying_customer = 'yes' then downloads else 0 end) as paying
  from ms_download_facts df
   left join ms_user_dimension ud using(user_id)
    left join ms_acc_dimension ad using(acc_id)
  group by 1
)t
where non_paying > paying
order by 1
```

### [Popularity Percentage](https://platform.stratascratch.com/coding/10284-popularity-percentage?code_type=1)  

```sql
with t as (
  select user1 as users, count(1) as cnt from facebook_friends group by 1
    union all
  select user2 as users, count(1) as cnt from facebook_friends group by 1
)
select 
  users,
  100.0 * sum(cnt) / (select count(1) from facebook_friends) as percentege
from t
group by 1
order by 1
```

### [Top 5 States With 5 Star Businesses](https://platform.stratascratch.com/coding/10046-top-5-states-with-5-star-businesses?code_type=1)  

```sql
with t as (
  select 
    state, name, sum(stars) as stars
  from yelp_business
  group by 1, 2
  ), f as (
  select 
    state,
    count(name) as cnt,
    rank() over(order by count(name) desc) as rnk
  from t where stars = 5
  group by 1
  )
select state, cnt 
from f where rnk <=5
order by cnt desc, state
```

### [Counting Instances in Text](https://platform.stratascratch.com/coding/9814-counting-instances-in-text?code_type=1)  

```sql
with t as (
  select 
    regexp_matches(contents, 'bull', 'g') as bull,
    regexp_matches(contents, 'bear', 'g') as bear
  from google_file_store
)
select 
  unnest(array['bull', 'bear']) as word,
  unnest(array[count(bull), count(bear)]) as cnt
from t
```

### [Host Popularity Rental Prices](https://platform.stratascratch.com/coding/9632-host-popularity-rental-prices?code_type=1)  

```sql
with t as (
  select distinct price, room_type, host_since, zipcode, number_of_reviews 
  from airbnb_host_searches
)
select 
  case
   when number_of_reviews = 0 then 'New'
   when number_of_reviews <= 5 then 'Rising'
   when number_of_reviews <= 15 then 'Trending Up'
   when number_of_reviews <= 40 then 'Popular'
  else 'Hot'
  end as  host_pop_rating,
  min(price) as minimum,
  avg(price) as mean,
  max(price) as maximum
from t
group by 1
```

### [Marketing Campaign Success [Advanced]](https://platform.stratascratch.com/coding/514-marketing-campaign-success-advanced?code_type=1)  

```sql
with t as (
  select 
    user_id,
    string_agg(distinct product_id::text, ', ') as product_id_ag
  from marketing_campaign
  group by 1
), t2 as (
  select t.user_id from t
   join (
    select 
      user_id,
      string_agg(distinct product_id::text, ', ') as product_id_ag
    from marketing_campaign
    where (user_id, created_at) in (
      select user_id, min(created_at) from marketing_campaign group by 1 )
    group by 1
) sq on t.user_id = sq.user_id and t.product_id_ag != sq.product_id_ag
)
select count(distinct user_id) from (
  select user_id
from marketing_campaign
where user_id in (select user_id from t2)
group by user_id
having count(distinct created_at) > 1 
 and count(distinct product_id) > 1
) foo
```

### [City With Most Amenities](https://platform.stratascratch.com/coding/9633-city-with-most-amenities?code_type=1)  

```sql
select city from (
 select 
  city,
  dense_rank() over (order by sum(array_length(string_to_array(amenities, ','), 1)) desc) r
 from airbnb_search_details
 group by 1
)t where r = 1
```

### [The Most Popular Client_Id Among Users Using Video and Voice Calls](https://platform.stratascratch.com/coding/2029-the-most-popular-client_id-among-users-using-video-and-voice-calls?code_type=1)  

```sql
with t as (
 select 
  client_id,
  user_id,
  1.0 * count(case when event_type in ('video call received',
                                       'video call sent',
                                       'voice call received',
                                       'voice call sent') 
              then 1 else null end) / count(*) as ratio
 from fact_events
 group by 1, 2
)
select client_id from t 
where ratio >= 0.5
group by 1
order by count(*) desc
limit 1
```

### [Top Percentile Fraud](https://platform.stratascratch.com/coding/10303-top-percentile-fraud?code_type=1)  

```sql
with prc (state, percentile) as (
 select state, percentile_disc(0.05) within group (order by fraud_score desc) 
 from fraud_score 
 group by 1
)
select fc.* from fraud_score fc
join prc on fc.state = prc.state and fraud_score >= percentile
```

### [Cookbook Recipes](https://platform.stratascratch.com/coding/2089-cookbook-recipes?code_type=1)  

```sql
with pages (page) as (
 select generate_series(
  (select min(page_number) from cookbook_titles),
  (select max(page_number) from cookbook_titles)
 )
)
select
 max(page) - 1 as left_page_number,
 max(case when page % 2 = 0 then title end) as left_title,
 max(case when page % 2 != 0 then title end)  as right_title
from (
 select 
  page / 2 as pn, page, title
 from pages p
 left join cookbook_titles ct on p.page = ct.page_number
)t
group by pn
order by pn
```

### [Retention Rate](https://platform.stratascratch.com/coding/2053-retention-rate?code_type=1)  

```sql
with y20 as (
 select distinct 
  account_id, 
  100.0 * sum((max_dt > '2020-12-31')::int) / count(*) as r20
 from (
  select 
   account_id, user_id,
   date, max(date) over (partition by user_id) as max_dt
  from sf_events
 )t
 where date_trunc('month', date) = '2020-12-01' 
 group by account_id
),
y21 as (
 select distinct 
  account_id, 
  100.0 * sum((max_dt > '2021-01-31')::int) / count(*) as r21
 from (
  select 
   account_id, user_id,
   date, max(date) over (partition by user_id) as max_dt
  from sf_events
 )t
 where date_trunc('month', date) = '2021-01-01' 
 group by account_id
)
select 
 y21.account_id, 
 r21 / r20 as retention
from y21, y20
where y21.account_id = y20.account_id
```