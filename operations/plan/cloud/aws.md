<a name="AWS"></a>
** AWS **

| Instance Type   | Client Transactions/s<sup>+</sup> | 95th Percentile (ms) | 99th Percentile (ms) | Object Size (bytes) | Object Count | Disk Size | Cost<sup>++</sup> |
| ---                       | ---       | ---                 | ---               | ---                  | ---           | ---     | ---       |
| i3.xlarge                 | 68,000    | 3.3                 | 5.4               | 1424                 | 240,000,000   | 950GB   | $0.312/hr |
| c3.4xlarge<sup>+++</sup>  | 66,000    | 4.1                 | 11                | 554                  | 240,000,000   | 2x160GB | $0.84/hr  |
| i2.xlarge                 | 36,000    | 3.5                 | 5.8               | 1424                 | 240,000,000   | 800GB   | $0.424/hr |

<sup>+</sup> As measured from the client, 50/50 read/update averaging under 1ms.  
<sup>++</sup> Price in us-west-2, at time of writing.  
<sup>+++</sup> All available ephemeral drives were utilized concurrently.  

{{#note}}
Some ephemeral drives require initialization before reaching maximum performance. You can read more on [Amazon's Documentation](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html#instance-store-volumes). 

Initialization is not done for these tests. The loading phase, as well as having multiple verification runs, acts as an effective initialization process.
{{/note}}

Your numbers may not neccessarily match these exactly, due to variance within AWS's environment.
