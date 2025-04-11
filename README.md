# Liocinema-Jotstar-Codebasics
total users of liocinema = 430752
total content = 1250

English	56
Hindi	424
Kannada	118
Malayalam	121
Marathi	68
Tamil	221
Telugu	242


Movie	900
Series	300
Sports	50


Action	167
Comedy	210
Crime	38
Documentaries	5
Drama	395
Family	79
Highlights	12
Horror	34
Live Matches	33
Romance	152
Thriller	125


25-34	52027
18-24	79813
35-44	32560
45+	19046

Tier 3	78587
Tier 1	41011
Tier 2	63848

Free	104992
Basic	53362
Premium	25092

Tier 3	182172567(total watch time)
Tier 1	221522138
Tier 2	258297683

Tier 1	41011(people total)
Tier 2	63848
Tier 3	78587


inactive user =303423	
total watch time =570654116
average watch time = 1880.72

active user = 127329	
total watch time = 91338272
average watch time = 717.34


SELECT * FROM liocinema_db.subscribers;
select age_group,count(age_group) from liocinema_db.subscribers
group by age_group;
select city_tier,count(city_tier) from liocinema_db.subscribers
group by city_tier;
select subscription_plan,count(subscription_plan) from liocinema_db.subscribers
group by subscription_plan;


SELECT 
    COUNT(CASE WHEN last_active_date IS NOT NULL THEN user_id END) AS active_users,
    COUNT(CASE WHEN last_active_date IS NULL THEN user_id END) AS inactive_users,
    COUNT(user_id) AS total_users,
    ROUND(
        (COUNT(CASE WHEN last_active_date IS NOT NULL THEN user_id END) * 100.0) / COUNT(user_id),
        2
    ) AS active_percentage,
    ROUND(
        (COUNT(CASE WHEN last_active_date IS NULL THEN user_id END) * 100.0) / COUNT(user_id),
        2
    ) AS inactive_percentage
FROM liocinema_db.subscribers;

SELECT 
    age_group,
    subscription_plan,
    COUNT(CASE WHEN last_active_date IS NOT NULL THEN user_id END) AS active_users,
    COUNT(CASE WHEN last_active_date IS NULL THEN user_id END) AS inactive_users,
    COUNT(user_id) AS total_users,
    ROUND(
        (COUNT(CASE WHEN last_active_date IS NOT NULL THEN user_id END) * 100.0) / COUNT(user_id),
        2
    ) AS active_percentage,
    ROUND(
        (COUNT(CASE WHEN last_active_date IS NULL THEN user_id END) * 100.0) / COUNT(user_id),
        2
    ) AS inactive_percentage
FROM liocinema_db.subscribers
GROUP BY age_group, subscription_plan
ORDER BY age_group, subscription_plan;

SELECT 
    subscribers.city_tier, 
    SUM(content_consumption.total_watch_time_mins) AS total_watch_time
FROM liocinema_db.subscribers 
INNER JOIN liocinema_db.content_consumption 
    ON content_consumption.user_id = subscribers.user_id
GROUP BY subscribers.city_tier
ORDER BY total_watch_time;

select city_tier,count(city_tier) as m from liocinema_db.subscribers
group by city_tier
order by  m;

select count(user_id) from (SELECT *
FROM liocinema_db.content_consumption cc
INNER JOIN liocinema_db.subscribers s
ON cc.user_id = s.user_id
WHERE s.last_active_date IS NULL) as joined_table;

SELECT COUNT(s.user_id) AS inactive_users_with_watch_time,sum(total_watch_time_mins)
FROM liocinema_db.subscribers s
INNER JOIN liocinema_db.content_consumption cc
ON s.user_id = cc.user_id
WHERE s.last_active_date IS NOT NULL;

SELECT 
    subscription_plan AS old_plan, 
    new_subscription_plan AS upgraded_plan, 
    COUNT(user_id) AS transition_count,
    ROUND(100.0 * COUNT(user_id) / SUM(COUNT(user_id)) OVER (), 2) AS transition_percentage
FROM liocinema_db.subscribers
WHERE new_subscription_plan IS NOT NULL
GROUP BY subscription_plan, new_subscription_plan
ORDER BY transition_count DESC;
select subscription_plan , count(subscription_plan) as count , city_tier from liocinema_db.subscribers
group by subscription_plan, city_tier
order by count
 ;






SELECT * FROM liocinema_db.contents;
select count(content_id) from liocinema_db.contents;
select language, count(language) from liocinema_db.contents
group by language
order by language; 

select content_type, count(content_type) from liocinema_db.contents
group by content_type
order by content_type;
select genre, count(genre) from liocinema_db.contents
group by genre
order by genre;




SELECT * FROM liocinema_db.content_consumption;
select count(user_id) as total_no from liocinema_db.content_consumption
;
SELECT 
    DATE_FORMAT(liocinema_db.subscribers.subscription_date, '%Y-%m') AS month, 
    COUNT(*) AS user_count
FROM liocinema_db.subscribers
GROUP BY month
ORDER BY month;

select count(user_id), sum(total_watch_time_mins) from liocinema_db.content_consumption;






WITH subscription_duration AS (
    -- Step 1: Calculate the duration under the initial plan
    SELECT
        user_id,
        subscription_plan AS plan,
        subscription_date AS start_date,
        IFNULL(plan_change_date, IFNULL(last_active_date, '2024-11-30')) AS end_date,
        TIMESTAMPDIFF(MONTH, subscription_date, IFNULL(plan_change_date, IFNULL(last_active_date, '2024-11-30'))) AS active_months
    FROM 
        liocinema_db.subscribers
    WHERE 
        subscription_date BETWEEN '2024-01-01' AND '2024-11-30'
        AND (last_active_date IS NULL OR last_active_date >= '2024-01-01')
        
    UNION ALL
    
    -- Step 2: Calculate the duration under the new plan after a change (if applicable)
    SELECT
        user_id,
        new_subscription_plan AS plan,
        plan_change_date AS start_date,
        IFNULL(last_active_date, '2024-11-30') AS end_date,
        TIMESTAMPDIFF(MONTH, plan_change_date, IFNULL(last_active_date, '2024-11-30')) AS active_months
    FROM 
        liocinema_db.subscribers
    WHERE 
        plan_change_date IS NOT NULL
        AND plan_change_date BETWEEN '2024-01-01' AND '2024-11-30'
),

plan_revenue AS (
    -- Step 3: Assign revenue per plan
    SELECT 
        'Free' AS plan, 0 AS monthly_rate
    UNION ALL
    SELECT 
        'Basic', 5
    UNION ALL
    SELECT 
        'Premium', 10
)

-- Step 4: Calculate total revenue and subscriber count
SELECT 
    sd.plan,
    COUNT(sd.user_id) AS subscriber_count,
    SUM(sd.active_months * pr.monthly_rate) AS total_revenue
FROM 
    subscription_duration sd
JOIN 
    plan_revenue pr ON sd.plan = pr.plan
WHERE 
    sd.active_months > 0
GROUP BY 
    sd.plan
ORDER BY 
    total_revenue DESC;



