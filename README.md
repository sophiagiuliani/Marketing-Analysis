# Marketing-Analysis

  This Analysis was made using data of the google merchandise store. The goal was to analyse the traffic sources and the performance of the marketing campaigns, by checking the session time, bounce rate and first session rate. After creating the dashboards, statistical tests such as correlation analysis, A/B test and regression analysis were perfomed in order to check the significance of the results, in order to make future decisions. To get the data needed, SQL was used.

**Analysis findings:**
  
  1. In general, customers spent more time per session on wednesdays and tuesdays. In subsequent sessions customers spent more time per session on fridays and thursdays. Sunday is the day with the least amount of time spent per session.
  2. The time spent online when the traffic source is referral is 121% higher than “Other”, which has the second highest score. The average order value is also higher for referral traffic source ($66.6). When comparing First Sessions with the Subsequent Sessions, the difference in general (all traffic sources) is 77%.
  3. The traffic source with the highest bounce rate is Direct, which means typing the website URL.
  5. Customers spent more time per session in Black Friday campaign and the least amount of time on New Year campaign. It is important to mention that The data do not follow a normal distribution, and the number of impressions is not enough to make assumptions.
  6. The highest Bounce Rate comes from Data Share Promo campaign. The second, Holiday V2.

**Suggestions:**

  1. Create a strategy to increase the engagement on sundays.
  3. Analyze the profile of customers who had more than one session.
       . high engagement but high bounce rate
  4. Keep investing in referral as a traffic source.
       . low bounce rate (especially on first sessions), high engagement.
  5. Investigate what is “Other” as a traffic source.
       . low bounce rate, high first session rate and engagement. 
  6. Create a strategy for next year Black Friday.
        . higher engagement


**Visualization:** 
[Looker Studio](https://www.example.com](https://lookerstudio.google.com/reporting/d8e974f9-cb73-432c-bccc-3547112fb496)


**SQL code:**

 WITH sessions AS (						
SELECT						
*,						
CASE WHEN (event_time - last_event_time) >= INTERVAL 30 MINUTE OR last_event_time IS NULL THEN 1 ELSE 0 END is_new_session						
FROM(						
SELECT						
user_pseudo_id,						
CAST(CONCAT(SUBSTRING(event_date,1,4), '-',SUBSTRING(event_date,5,2),'-',SUBSTRING(event_date,7,2))AS DATE) AS event_date,						
campaign,						
TIMESTAMP_MICROS(event_timestamp) AS event_time,						
LAG(TIMESTAMP_MICROS(event_timestamp)) OVER (PARTITION BY user_pseudo_id ORDER BY TIMESTAMP_MICROS(event_timestamp)) AS last_event_time,						
category						
FROM tc-da-1.turing_data_analytics.raw_events)),						
						
global_sessions AS (						
SELECT						
user_pseudo_id,						
event_date,						
event_time,						
last_event_time,						
event_time - LAG(event_time) OVER (PARTITION BY global_session_id ORDER BY event_time) AS timediff,						
global_session_id,						
user_session_id,						
campaign,						
category						
FROM(						
SELECT						
user_pseudo_id,						
event_date,						
event_time,						
last_event_time,						
SUM(is_new_session) OVER (ORDER BY user_pseudo_id, event_time) AS global_session_id,						
SUM(is_new_session) OVER (PARTITION BY user_pseudo_id ORDER BY event_time) AS user_session_id,						
campaign,						
category						
FROM sessions)),						
						
time_difference AS (						
SELECT						
user_pseudo_id,						
event_date,						
SUM(timediff) AS timestamotimediff,						
global_session_id,						
user_session_id,						
campaign,						
category						
FROM global_sessions						
GROUP BY 1,2,4,5,6,7),						
						
fix_time_nulls AS (						
SELECT						
user_pseudo_id,						
event_date,						
SUM(COALESCE(time_seconds, 0)) AS time_seconds,						
global_session_id,						
user_session_id,						
ccampaign,						
category						
FROM(						
SELECT						
user_pseudo_id,						
event_date,						
EXTRACT(HOUR FROM timestamotimediff) * 3600 + EXTRACT(MINUTE FROM timestamotimediff) * 60 + EXTRACT(SECOND FROM timestamotimediff) AS time_seconds,						
global_session_id,						
user_session_id,						
FIRST_VALUE(campaign) OVER (PARTITION BY global_session_id ORDER BY campaign DESC) AS ccampaign,						
category						
FROM time_difference)						
GROUP BY user_pseudo_id, event_date, global_session_id, user_session_id, category, ccampaign)						
						
SELECT						
user_pseudo_id,						
event_date,						
time_seconds,						
global_session_id,						
user_session_id,						
ccampaign,						
category,						
bounce_session AS is_bounce_session,						
CASE WHEN first_session = 1 THEN 1 ELSE 0 END AS is_first_session						
FROM(						
SELECT						
*,						
CASE WHEN time_seconds = 0 THEN 1 ELSE 0 END AS bounce_session,						
ROW_NUMBER() OVER (PARTITION BY user_pseudo_id ORDER BY global_session_id) AS first_session						
FROM fix_time_nulls)						
ORDER BY user_pseudo_id, event_date						
