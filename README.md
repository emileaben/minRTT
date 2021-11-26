# minRTT
minimum RTT dataset and viz from RIPE Atlas

This is the public documentation for the minRTT dataset and visualisation that we make at the RIPE NCC

# TODOs
 - document table locations
 - document how to deal with data quality (probe geoloc, rtt lies)

# SQL query running daily in BigQuery
This is the SQL query we run in [our BigQuery environment](https://github.com/RIPE-NCC/ripe-atlas-bigquery) which populates our dataset:

```sql
-- we'll buid the query in 3 layers

-- let's start by joining two datasets:
--  * min_rtt: min. RTT perceived by Atlas for each IP address we see
--  * ip_matches: Network information coming from [RIS](https://ris.ripe.net)
-- we end up with min. RTT per network
with minrtt_origin as (
    with
    m as ( select * from `prod-atlas-project.atlas_utils.min_rtt`), -- this is a larger minRTT dataset, with no information about nets
    ip as ( select * from `ripencc-testing-env.ris.ip_matches`)     -- RIS information

    select prb_id, origin, af, m.day, min(min_rtt) as min_rtt, count(distinct ip_addr) as ip_count, sum(m.hop_count) as samples, protoc
    from   m
    join   ip
    on m.day = date(ip.day)
    and m.ip_addr = ip.ip
    and m.day = DATE_SUB(@run_date, INTERVAL 1 DAY)
    and origin is not null
    and origin not like '%{%' -- drop AS-sets like {1,2,3}

    group by prb_id, origin, af, m.day, protoc
),

-- the following adds the `nets` array
-- which is just a split-by-comma operation
-- "1,2,3" --> ["1", "2", "3"]
-- we also account for multi-origin announcements (MOAs)
split_nets as (
    select *,
    -- CASE for shielding from exceptions like:
    -- ix;270;InterLAN;Bucharest,-Arad,-Cluj-Napoca,-Constanta,-Craiova,-Iasi,-Suceava,-Timisoara;RO
    -- (it has a comma, and is not a MOA)
    CASE
        WHEN CONTAINS_SUBSTR(origin, ',') AND NOT CONTAINS_SUBSTR(origin, ';') THEN split(origin, ',') -- MOAs: only strings like "1,2,3"
        ELSE [origin] --single-item array
    END AS nets
    FROM minrtt_origin
    WHERE day = DATE_SUB(@run_date, INTERVAL 1 DAY)
)

-- at this stage we've got the `nets` array.
-- because we're doing a group-by we'll need to 
-- derive some aggregates (min, samples, etc...)
select * except (nets, _net, min_rtt, ip_count, origin, samples),
    CASE
        WHEN CONTAINS_SUBSTR(any_value(_net), ';') THEN concat('ix-', split(any_value(_net), ';')[OFFSET(1)])
        ELSE any_value(_net)
    END AS net,
    any_value(origin) as original_net,
    min(min_rtt) as min_rtt, sum(ip_count) as ip_count, sum(samples) as samples
-- `_net` will be our raw net field, we'll prettify to `net` in the CASE statement
from split_nets, unnest(nets) as _net
group by prb_id,_net,af,protoc,day
```

# Publicly available datasets
We currently (Nov 2021) host this on GCP and data is available at:
 * [Browsable listing by network](https://console.cloud.google.com/storage/browser/ripencc_public/rd/min-rtt-by-net?pageState=(%22StorageObjectListTable%22:(%22f%22:%22%255B%255D%22))&project=prod-atlas-project&prefix=&forceOnObjectsSortingFiltering=false)
   - we kindly ask users to fetch the data from the CDN: `https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-net/latest/<net>.json` where net can be an ASN or an IXP id as seen in PeeringDB.
   - [Example for AS3333](https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-net/latest/3333.json)
 * [Browsable listing by probe](https://console.cloud.google.com/storage/browser/ripencc_public/rd/min-rtt-by-prb_id?pageState=(%22StorageObjectListTable%22:(%22f%22:%22%255B%255D%22))&project=prod-atlas-project&prefix=&forceOnObjectsSortingFiltering=false)
   - we kindly ask users to fetch the data from the CDN: `https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-prb_id/latest/<prb_id>.json`
   - [Example for probe #1](https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-prb_id/latest/1.json)

(these URLs will likely change! We want to get rid of the 'origin' in it and start using more descriptive 'net' as a name for ASNs or IXPs)
  
# Observable notebooks showcasing these datasets
Obeservable viz around this:
  * https://observablehq.com/@ripencc/atlas-latency-worldmap
  * https://observablehq.com/@ripencc/atlas-probe-neighbourhood
  * https://observablehq.com/@aguformoso/atlas-probe-proximity-to-ixps. (which IXPs are we 'covering' with RIPE Atlas), allows for tweaking of the parameters for 'covering'). Idea here is that we could leverage this to get more IXPs covered. Might be a campaign for probe placement we run in the near future.

# Describe success stories

# Other docs (ripe labs etc.)
