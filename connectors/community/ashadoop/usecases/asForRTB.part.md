
### Ad Targeting and Real-Time Bidding (RTB)
The figure below shows the real-time bidding ecosystem. Supply Side Platforms (SSPs) and Demand Side Platforms (DSPs) use Aerospike in-memory key value store to look up and store attributes pertaining to userids.  SSP and DSP platform providers use Hadoop for collecting web server log data and update the userid attributes periodically in batch mode to Aerospike.



<center>
{{#figure "" "Aerospike in Real Time Bidding" size="large" }}
<img src="/docs/connectors/assets/images/AerospikeInRTB.png" >
{{/figure}}
</center>


The diagram above shows all what happens in the 100 ms window from the instant a user lands on a publisher’s site and the Content Delivery Network serves a display ad. This represents the total time budget for requesting bids from all programmatic buyers for that spot, allowing bidders time to decide their bids and then select the winning bid based on criteria other than just the bid price, displaying the bid, collecting post ad-display information to feed it back into the system, the programmatic buying part has a typical SLA of 10 ms.

A typical online display advertising company may be processing 50 TB of data monthly, 100 to 500 million direct observations/hour, 2 million requests per minute handling peaks of 10 to 12 million requests per minute. They need consistent sub-millisecond writes, 4K to 5K reads per sec, tenth of ms for empty reads (no data). Consistent performance is extremely crucial. The real-time systems running SSD based Aerospike clusters enable the ad selection algorithms to run with tight latency numbers and allow programmatic buyers to target ads with criteria such as geo-targeting, day parting (time of day), costs-budgets-floors-max$$ per impression, segment cookies and other third party data.

