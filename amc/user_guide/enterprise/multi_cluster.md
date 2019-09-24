---
title: Multi Cluster View
description: Learn how to switch to different clusters in the Enterprise Edition management console.
---
Multi-Cluster view in AMC is designed to monitor multiple clusters within an enterprise through a single screen, available only for the users with enterprise licensed software.

Using this feature an enterprise admin/user can monitor multiple clusters within the same browser session. 


Multi-Cluster view setup:

Multi-Cluster view is supported by Aerospike Server version 3.6.0 and above for enterprise AMC users only.


Accessing Multi-Cluster view:
From the landing page a user needs to provide the seed node ip and check the multi-cluster view check-box and click on connect as shown below.

<img src="/docs/amc/assets/images/E_landing.png" alt="landingpage" width="800">

From Setting option user can click on Multi-Cluster view to access this feature.

<img src="/docs/amc/assets/images/E_setting.png" alt="AMC_Setting" width="800">

Details of Multi-Cluster view:
<table border="0">


        <tr>
                <td>
                        <img src="/docs/amc/assets/images/1.png" alt="3" width="24">
                </td>

                <td>
                      On Mouse-over green & red dots, user can see the node details- Node IP, Port and Node Status information.
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/2.png" alt="3" width="24">
                </td>

                <td>
                      On hovering over the cluster circles, user can see the configured namespaces
                </td>
        </tr>
        <tr>
 <td>
                        <img src="/docs/amc/assets/images/3.png" alt="3" width="24">
                </td>

                <td>
                      User can add a new cluster manually.
                </td>
        </tr>

</table>

{{#note}}
The user needs to click on any cluster to go to the dashboard for that particular cluster.
{{/note}}


A user can add clusters manually  through the Add Cluster option.
Please refer to the screenshots below to see how to add a cluster.
After Clicking on the Add Cluster option, a Cluster Seed popup will appear prompting the user to enter the fields like Host IP and Port. The user needs to provide cluster IP and check the multi-cluster view option.

<img src="/docs/amc/assets/images/E_nonxdrseed.png" alt="nonxdr_cluster" width="800">

After connecting to another cluster, the user can see all the added clusters in multi-cluster view wherein a user can monitor multiple clusters in a single view.


