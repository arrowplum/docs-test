<a name="IBM"></a>
** IBM Cloud **

| Instance Type   | Client Transactions/s<sup>+</sup> | 95th Percentile (ms) | 99th Percentile (ms) | Object Size (bytes) | Object Count | Disk Size | Cost<sup>++</sup> |
| ---                      | ---       | ---                 | ---               | ---                  | ---           | ---     | ---       |
| Xeon E3 1270v3<br>1Gig Networking<br> VM to Baremetal | 75,000  | 2.7  | 4.3   | 500                  | 240,000,000   | 1.2TB   | $415/mo ($0.576/hr) | 
| Xeon E3 1270v3<br>1Gig Networking<br> Baremetal to Baremetal | 130,000  | 3.4  | 5.5  | 500           | 240,000,000   | 1.2TB   | $415/mo ($0.576/hr) |
| Xeon E3 1270v3<br>1Gig Networking<br> Baremetal to Baremetal | 30,000   | 3.0  | 4.6  | 3000          | 240,000,000   | 1.2TB   | $415/mo ($0.576/hr) |
| 2x Xeon E5 2620v4<br>10Gig Networking<br>Baremetal | 222,000 | 1.7   | 3.2    | 500                   | 240,000,000   | 1.2TB   | $1109/mo ($1.54/hr) |
| 2x Xeon E5 2620v4<br>10Gig Networking<br>Baremetal <sup>+++</sup> | 100,000   | 2.6  | 3.4   | 3000   | 240,000,000   | 1.2TB   | $1109/mo ($1.54/hr) |

<sup>+</sup> As measured from the client, 50/50 read/update averaging under 1ms.  
<sup>++</sup> Price in SJC01/SJC04, at time of writing. System, RAM, Networking and Disk are all separate billable items, shown as a combined sum. Servers were on monthly provisioning, with hourly costs extrapolated using 30-day months.  
<sup>+++</sup> SSD Device overloaded and testing was terminated early. Testing proceeded at a faster pace than the device was capable of while still able to maintain latency profile. This caused defragmentation to fall behind and exhausted free blocks, therefore Aerospike Server entered into [`stop_writes`](/docs/reference/metrics/#stop_writes). Read more about [stop-writes](https://discuss.aerospike.com/t/faq-what-are-expiration-eviction-and-stop-writes/2311) and how it relates to [defragmentation](/docs/operations/manage/storage#aerospike-s-automatic-storage-reclamation-processes). 

Your numbers may not neccessarily match these exactly, due to variance within IBM's environment.
