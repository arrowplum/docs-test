---
title: Introduction - Aerospike REST Client
description: Use the Aerospike REST Client to communicate with an Aerospike Cluster via RESTful requests.
---

Use the Aerospike REST Client to communicate with an Aerospike Cluster via RESTful requests.

The REST Client is a Java application which can be built into a `.war` file and deployed to a WAR container such as Tomcat.

###Code

This example imports utilizes the Python Requests Library to create, update and delete a record via the REST Client.

```python
import requests

REST_BASE = 'http://localhost:8080/v1'
KVS_ENDPOINT = REST_BASE + '/kvs'

# Components of the Key
namespace = 'test'
setname = 'users'
userkey = 'bob'

record_uri = '{base}/{ns}/{setname}/{userkey}'.format(
    base=KVS_ENDPOINT, ns=namespace, setname=setname, userkey=userkey)

# The content to be stored into Aerospike.
bins = {
    'name': 'Bob',
    'id': 123,
    'color': 'Purple',
    'languages': ['Python', 'Java', 'C'],
}


# Store the record
res = requests.post(record_uri, json=bins)

# Get the record
# It is a map: {
#    'bins': {},
#    'generation': #,
#    'ttl': #
# }
response = requests.get(record_uri)
print("*** The Original Record ***")
print(response.json())

# Change the value of the 'color' bin
update_bins = {'color': 'Orange'}
requests.patch(record_uri, json=update_bins)

# Get the updated Record. Only the 'color' bin has changed
response = requests.get(record_uri)
print("*** The updated Record ***")
print(response.json())

# Replace the record with a new version
replacement_bins = {'single': 'bin'}
requests.put(record_uri, json=replacement_bins)

# Get the new Record.
response = requests.get(record_uri)
print("*** The Replaced Record ***")
print(response.json())

# Delete the record.
response = requests.delete(record_uri)

# Try to get the deleted . We will receive a 404.
response = requests.get(record_uri)

print('*** The response code for a GET on a non existent record is {} ***'.format(response.status_code))

# The response also includes a JSON error object
print("*** The Error object is: ***")
print(response.json())

```

<div style="text-align: center;">
<a class="button primary" href="/docs/client/rest/start/index.html">GET STARTED</a>
</div>
