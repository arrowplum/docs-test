<a name="Azure"></a>
** Azure **

| Instance Type   | Client Transactions/s<sup>+</sup> | 95th Percentile (ms) | 99th Percentile (ms) | Object Size (bytes) | Object Count | Disk Size | Cost<sup>++</sup> |
| ---             | ---       | ---                 | ---               | ---                | ---           | ---   | ---       |
| Standard_Gs1    | 31,000    | 3.5                 | 9.1               | 13<sup>+++</sup>   | 240,000,000   | 56GB  | $0.61/hr  |
| Standard_L4s    | 41,000    | 1.2                 | 5.8               | 1500               | 240,000,000   | 678GB | $0.344/hr |


<sup>+</sup> As measured from the client, 50/50 read/update averaging under 1ms.  
<sup>++</sup> Price in West US, at time of writing.  
<sup>+++</sup> Low object size is due to fitting 240m objects into a mere 56GB disk. Larger object sizes can be used with the following optimzations:  
&nbsp;&nbsp;&nbsp;- shorten set name from 9 characters  
&nbsp;&nbsp;&nbsp;- Enable [single-bin](https://www.aerospike.com/docs/reference/configuration/index.html#single-bin) for the namespace. This will make 28 additional bytes available.  
&nbsp;&nbsp;&nbsp;- 3 additional bytes are available for Integer or Float types.  
&nbsp;&nbsp;&nbsp;This results in up to an additional 39 bytes for data per record without impacting storage space (using a single character set name).


Your numbers may not neccessarily match these exactly, due to variance within Azure's environment.
