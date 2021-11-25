# minRTT
minimum RTT dataset and viz from RIPE Atlas

This is the public documentation for the minRTT dataset and visualisation that we make at the RIPE NCC

# Describe process how we create (SQL query), table locations, decisions on how to deal with as-sets / moas etc. / deal with data quality (probe geoloc, rtt lies)

# Publicly available datasets
We currently (Nov 2021) host this on GCP and data is available at:
 * https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-origin/latest/<origin-id\>.json
 * https://rd-gcp-cdn.ripe.net/rd/min-rtt-by-prb_id/latest/<prb-id\>.json 

(these URLs will likely change! We want to get rid of the 'origin' in it and start using more descriptive 'net' as a name for ASNs or IXPs)
  
# Describe observable stuff
Obeservable viz around this:
  * https://observablehq.com/@ripencc/atlas-latency-worldmap
  * https://observablehq.com/@ripencc/atlas-probe-neighbourhood
  * https://observablehq.com/@aguformoso/atlas-probe-proximity-to-ixps. (which IXPs are we 'covering' with RIPE Atlas), allows for tweaking of the parameters for 'covering'). Idea here is that we could leverage this to get more IXPs covered. Might be a campaign for probe placement we run in the near future.

# Describe success stories

# Other docs (ripe labs etc.)
