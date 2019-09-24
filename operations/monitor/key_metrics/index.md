---
title: Key Metrics to Monitor
description: Key Metrics to monitor in Aerospike
styles:
  - http://cdn.datatables.net/1.10.19/css/jquery.dataTables.css
scripts:
  - http://cdn.datatables.net/1.10.19/js/jquery.dataTables.js
---
{{#markdown}}
A recommended set of metrics to monitor are provided below. Metrics are
divided into {{metrics.categories.length}} categories, each category
indicates a common system component that may cause the metric to
report a alert/critical value. The categories are as follows:
{{/markdown}}

<ul>
  {{#metrics.categories}}
    <li><strong>{{name}}</strong>: {{description}}</li>
  {{/metrics.categories}}
</ul>

{{#markdown}}
In addition to monitoring statistics below operations should also monitor Linux
vitals such as free disk space, free RAM, swapping, etc.
{{/markdown}}

{{#markdown}}
### Metric Table

{{#info}}
See the [Metric Reference](/docs/reference/metrics) for a complete metric list
as well as information on how to access the metrics from the command line.
{{/info}}
{{/markdown}}

<table id="metrics-table">
  <thead>
    <tr>
      <th>Name</th>
      <th>Usage</th>
    </tr>
  </thead>
  <tbody>
    {{#each metrics.contexts}}
        {{#filter metrics "item['kpi'] == true"}}
        <tr>
            <td>
            {{name}}<br>
            {{#cumulative}}<sup(Cumulative)</sup><br>{{/cumulative}}
        {{#category}}<strong>Category:</strong> {{lowercase .}}<br>{{/category}}
        {{#location}}<strong>Location:</strong> {{lowercase .}}<br>{{/location}}
            </td>
            <td>{{#markdown}}{{usage}}{{/markdown}}</td>
        </tr>
        {{/filter}}
    {{/each}}
  </tbody>
</table>

<script>
   $("#metrics-table").dataTable( {
    "oLanguage": {
        "sSearch": ""
      },
     paging: false,
     columnDefs: [
       {
         targets: [0, 1],
         orderData: [1, 0],
       },
     ],
   });

   $("#monitoring-solutions-table").dataTable( {
     paging: false,
     bInfo: false,  // Removes number of records form bottom
     bFilter: false, // Removes search box at top right
   });

  setTimeout(function(){
    $('#metrics-table_filter label input').attr('placeholder', 'Search...');
  }, 1000);
</script>
