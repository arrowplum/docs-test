---
title: Introduction - Node.js Client
description: Use the Aerospike Node.js client with the Aerospike database using Linux Redhat/CentOS 6.x, Ubuntu 12+, Debian 6+, etc. or Mac OS X platforms. 
---

Use the Aerospike Node.js client to build Node.js applications to store and
retrieve data from an Aerospike cluster.

The client is compatible with Node.js v4.x (LTS), v6.x (LTS), v8.x (LTS), and
v10.x. It supports the following operating systems: CentOS/RHEL 6/7, Debian
7/8, Ubuntu 12.04/14.04/16.04, as well as many Linux destributions compatible
with one of these OS releases. macOS is also supported.

### Code

The following is very simple example how to create, update, read and remove a
record using the Aerospike database.

```js
const Aerospike = require('aerospike')

let config = {
  hosts: '192.168.33.10:3000'
}
let key = new Aerospike.Key('test', 'demo', 'demo')

Aerospike.connect(config)
  .then(client => {
    let bins = {
      i: 123,
      s: 'hello',
      b: Buffer.from('world'),
      d: new Aerospike.Double(3.1415),
      g: new Aerospike.GeoJSON({type: 'Point', coordinates: [103.913, 1.308]}),
      l: [1, 'a', {x: 'y'}],
      m: {foo: 4, bar: 7}
    }
    let meta = { ttl: 10000 }
    let policy = new Aerospike.WritePolicy({
      exists: Aerospike.policy.exists.CREATE_OR_REPLACE
    })

    return client.put(key, bins, meta, policy)
      .then(() => {
        let ops = [
          Aerospike.operations.incr('i', 1),
          Aerospike.operations.read('i'),
          Aerospike.lists.append('l', 'z'),
          Aerospike.maps.removeByKey('m', 'bar')
        ]

        return client.operate(key, ops)
      })
      .then(result => {
        console.log(result.bins)   // => { c: 4, i: 124, m: null }

        return client.get(key)
      })
      .then(record => {
        console.log(record.bins) // => { i: 124,
                                 //      s: 'hello',
                                 //      b: <Buffer 77 6f 72 6c 64>,
                                 //      d: 3.1415,
                                 //      g: '{"type":"Point","coordinates":[103.913,1.308]}',
                                 //      l: [ 1, 'a', { x: 'y' }, 'z' ],
                                 //      m: { foo: 4 } }
      })
      .then(() => client.close())
      .catch(error => {
        client.close()
        return Promise.reject(error)
      })
  })
  .catch(error => console.log(error))
```

<br/>

<div class="text-center">
<a class="button primary" href="/docs/client/nodejs/start">GET STARTED</a>
</div>
