<a name="GCP"></a>
** GCP **

| Instance Type   | Client Transactions/s<sup>+</sup> | 95th Percentile (ms) | 99th Percentile (ms) | Object Size (bytes) | Object Count | Disk Size | Cost<sup>++</sup> |
| ---             | ---       | ---                 | ---               | ---   | ---           | ---    | ---                      |
| n1_standard_8   | 82,000    | 3.2                 | 5.9               |  768  | 240,000,000   | 375GB  | $0.492/hr<sup>+++</sup>  |


<sup>+</sup> As measured from the client, 50/50 read/update averaging under 1ms.  
<sup>++</sup> Price in us-central1-a, at time of writing.  
<sup>+++</sup> Cost includes $0.112/hr for a single local ssd.  

Your numbers may not neccessarily match these exactly, due to variance within Google's environment.
