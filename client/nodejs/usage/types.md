---
title: Data Types
description: The Aerospike Node.js client supports integers, strings, and byte arrays for bin and key values. 
---

The Aerospike Node.js client supports integers, strings, and byte arrays for bin and key values.

### Integers

Aerospike supports storing 64-bit integers. You can create integers using JavaScript Number.

Aerospike does not support floating point numbers. Numbers containing float values are not stored in the database.

**Examples**

To use integers for key values:

```js
var key = new Aerospike.Key('test', 'demo', 123)
```

To use integers for record bin values:

```js
let bins = {
  id: 12345678,
  age: 32
}
```

### Strings

Aerospike supports storing NULL-terminated strings. You can create strings using JavaScript String.

**Examples**

To use strings for key values:

```js
let key = new Aerospike.Key('test', 'demo', 'ABC')
```

To use strings for record bin values:

```js
let bins = {
  first_name: 'John',
  last_name: 'Doe'
}
```

### Byte Arrays

Aerospike supports storing byte arrays. You can create byte arrays using JavaScript Uint8Array or Node.js Buffer.

**Examples**

To use the `Buffer` byte array for key values:

```js
let key = new Aerospike.Key('test', 'demo', Buffer.from([0xa, 0xb, 0xc]))
```

To use the `Buffer` byte array for record bin values:

```js
let bins = {
  avatar: Buffer.from([0xa, 0xb, 0xc]),
  password: Buffer.from([0xa, 0xb, 0xc])
}
```

To use the `Uint8Array` byte array for record bin values:

```js
let bins = {
  avatar: new Uint8Array([0xa, 0xb, 0xc]),
  password: new Uint8Array([0xa, 0xb, 0xc])
}
```
