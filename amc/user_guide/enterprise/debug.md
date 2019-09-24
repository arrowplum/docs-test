---
title: Debug
description: Learn how to use debug feature on the Enterprise Edition management console.
---
In an attempt to do a root cause analysis, end user can turn ON the debug feature **(Available in AMC version => 3.6.6)** and reproduce the issue for the logs and stats to be collected in the log file on the AMC server. Log files can then be used for investigation and fact finding. After turning the debug feature ON, logging will begin and write to a file at the AMC log location. Every activity done by the user is then intelligently captured and logged in the log file that will enable smart investigation within a quick turn around time.

Default time to record activity is 10 minutes, or user can set the time according to requirements.

Please refer to the steps below to use debug feature:
<table border="0">
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/1.png" alt="1" width="24">
                </td>
                <td>
                       Click on Settings icon.
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/2.png" alt="2" width="24">
                </td>
                <td>
                       Turn the logging option ON.
               </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/3.png" alt="3" width="24">
                </td>
                <td>
                      Set time and click on START.
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/4.png" alt="4" width="24">
                </td>
                <td>
                      Repeat the steps reqiured to reproduce the observed issues.                   
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/5.png" alt="5" width="24">
                </td>
                <td>
                      Queries will be recorded in the log file.
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/6.png" alt="6" width="24">
                </td>
                <td>
                      Once the process is complete, logging option can be turned OFF.
                </td>
        </tr>
        <tr>
                <td>
                        <img src="/docs/amc/assets/images/7.png" alt="7" width="24">
                </td>
                <td>
                      User can click on restart (when logging is on) to start recording queries from the start.
                </td>
        </tr>
</table>
User can find log file from this location: /var/log/amc/aerospike-amc.log

<img src="/docs/amc/assets/images/E_debug1.png" alt="Debug" width="800">

<img src="/docs/amc/assets/images/E_debug.png" alt="Debug" width="800">

Note: As all queries and responses are recorded, log files generated after enabling debug feature may get larger over the time.
