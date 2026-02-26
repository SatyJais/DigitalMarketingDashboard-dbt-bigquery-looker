```sql
with
    dos_mapped as (select * from {{ ref("Discussing_Options_Ads_Mapped") }}),
    leads_mapped as (select * from {{ ref("Leads_Ads_Mapped") }}),
    ats_mapped as (select * from {{ ref("AT_Ads_Mapped") }}),
    google_metrics as (select * from {{ ref("Google_Ads_Metrics") }}),
    daily_dos as (
        select date, device, count(user_id) as dos
        from dos_mapped
        where channel = 'Google'
        group by date, device
        order by date desc

    ),
    daily_leads as (
        select date, device, count(user_id) as leads
        from leads_mapped
        where channel = 'Google'
        group by date, device
        order by date desc

    ),
    daily_ats as (
        select date, device, count(user_id) as ats
        from ats_mapped
        where channel = 'Google'
        group by date, device
        order by date desc
    ),
    daily_google_metrics as (
        select
            segments_date,
            segments_device,
            sum(metrics_impressions) as impressions,
            sum(metrics_clicks) as clicks,
            sum(metrics_conversions) as conversions,
            sum(cost) as spends
        from google_metrics
        group by segments_date, segments_device
        order by segments_date desc

    ),
    combined as (
        select
            segments_date,
            segments_device,
            impressions,
            clicks,
            conversions,
            spends,
            coalesce(leads, 0) as leads,
            coalesce(dos, 0) as dos,
            coalesce(ats, 0) as ats
        from daily_google_metrics gm
        left join
            daily_leads dl
            on gm.segments_date = dl.date
            and gm.segments_device = dl.device
        left join
            daily_dos dd
            on gm.segments_date = dd.date
            and gm.segments_device = dd.device
        left join
            daily_ats da
            on gm.segments_date = da.date
            and gm.segments_device = da.device
        order by segments_date desc

    )

select *
from combined

```
