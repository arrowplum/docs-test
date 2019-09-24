---
title: bytes
description: Learn how Aerospike's bytes module allows you to create and manipulate a byte array, similar to a database BLOB or network byte buffer.
---

### Usage

The bytes module allows you to create and manipulate a byte array, similar to a database BLOB or network byte buffer.

The module contains the definition for the Bytes type with functions for easily manipulating bytes directly, leading to efficient packed representations of data.

A limitation of Lua is there is a single representation for numbers, so it is difficult to distinguish between integers of varying widths (8-bit, 16-bit, 32-bit and 64-bit). 

The byte array in a bytes object has a known size at any given moment. Attempting to read or write outside of the size will result in failure of the read or write operation. The object can be resized efficiently without constructing a new object.

A bytes object can be placed within an Aerospike list or map object.

Special care must be taken when packing 8-byte integers, and 4-byte unsigned values into the byte array. See the sections below regarding those functions.

Please note in all sections that byte array indexes are in Lua-style 1-indexed notation - that is, the first byte in the array is byte 1. This structure is similar to a C byte buffer, and a frequent mistake is to use C-style 0-based indexes.

#### Creating a Bytes Object
To create an empty bytes object, call the bytes() function. The parameter is the initial capacity of the byte array, in bytes. 

For example, to create an 18-byte array, you would use the following command from the Lua console:

```lua
> local b = bytes(18)
Bytes()
```

#### Accessing Bytes in the Array

You can access the bytes in the byte array like a Lua Table. The commands below access the first byte of the array, sees that it is nil since not yet set, then setting that byte to 5 and then accessing the value.

```lua
> b[1]
nil
  
> b[1] = 5
> b[1]
5
```

#### Storing and Retrieving Bytes

Individual functions allow you to pack and unpack bytes. In the console example below, we store 0x5555 as the first two bytes in the array, followed by 0x66778899 at offset 3. This leaves us with a byte array containing: 55 55 66 77 88 99 and we can retrieve the bytes individually, as in the example below, we retrieve just byte 4 (0x77).

```lua
> bytes.set_int16(b, 1, 0x5555)
> bytes.set_int32(b, 3, 0x66778899)
> bytes[4]
119
```

#### Encoding Type

The Bytes object may have an associated type, which represents the form of binary serialization.

The types are usually language specific and are provided as a convenience. As an example, if you use the Java type, then it implies the data was serialized using the default Java serializer. This gives clients that understand Java serialization, the ability to serialize and deserialize the data semi-automatically.

The supported types are not exhaustive. Ideally, you can use the Generic type, and pack bytes in the beginning to provide system specific information, allowing you to serialize using other schemes, such as ProtocolBuffers, JSON, or XML.

| Value | Type | Description |
| --- | --- | --- |
| 4 | Generic | (default) An untyped byte array. |
| 7 | Java | A Java Serialized object, created using Java default serializer. |
| 8 | C# | A C# Serialized object, created using C# default serializer. |
| 9 | Python | A Python Pickled object, created using Pythos default pickler. |
| 10 | Ruby | A Ruby serialized object. |
| 11 | PHP | A PHP serialized object, created using PHP default serializer. |

### Functions

-------------------------------------------------------------------------------
#### bytes()

Creates a new Bytes instance.

```lua
function bytes(n: Integer?): Bytes
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`n` – (optional) the initial size of the byte buffer.
    </ul>
  </dd>
  <dt>Returns
  <dd>The initialized `Bytes` object.
</dl>

Examples:

To create an empty Bytes with zero-length array:

```lua
> local b = bytes()
Bytes()
```

To create an empty Bytes of a specific size:

```lua
> local b = bytes(18)
Bytes()
```


-------------------------------------------------------------------------------
#### bytes.size()

Get the size of Bytes `b`.

```
function bytes.size(b: Bytes): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the size of.
    </ul>
  </dd>
  <dt>Returns
  <dd>The number of bytes in `Bytes`
</dl>

Example:

```lua
> local b = bytes(4)
Bytes()
> bytes.size(b)
4
```

-------------------------------------------------------------------------------
#### bytes.set_size()

Set the size of Bytes `b`. If the new size is larger than the current size, the object is enlarged to the specified size, the additional bytes are added to the end of the byte array and initialized to 0. Internally a realloc() call will change the memory size. If the new size of the byte array is smaller, bytes are truncated from the end of the byte array, but memory is not freed.

```lua
function bytes.set_size(b: Bytes, size: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the size of.
      <li>`size` – the size to set `b` to.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_type()

Get the [encoding type](#encoding-type) of Bytes `b`.

```
function bytes.get_type(b: Bytes): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` object.
    </ul>
  </dd>
  <dt>Returns
  <dd>The encoding type of bytes in `Bytes`
</dl>

Example:

```lua
> local b = bytes(100)
Bytes()
> bytes.get_type(b)
4
```

-------------------------------------------------------------------------------
#### bytes.set_type()

Set the [encoding type](#encoding-type) of Bytes `b`.

```lua
function bytes.set_type(b: Bytes, type: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the size of.
      <li>`type` – the [encoding type](#encoding-type).
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>

Example:

```lua
> local b = bytes(100)
Bytes()
> bytes.set_type(b, 4)
```


-------------------------------------------------------------------------------
#### bytes.get_string()

 Get string of length `l` from Bytes `b`.

```lua
function bytes.get_string(b:Bytes, i: Integer, l: Integer): String
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to get the bytes from. 
      <li>`i`  - Offset from which the bytes should be read. 
      <li>`l`  - Total length of string from offset `i` to be read from `b`. 
    </ul>
   </dd>
</dl>


-------------------------------------------------------------------------------
#### bytes.get_bytes()

 Get bytes of given length `l` from Bytes `b`.

```lua
function bytes.get_bytes(b:Bytes, i: Integer, l: Integer): Bytes
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to get the bytes from. 
      <li>`i`  - Offset from which the bytes should be read. 
      <li>`l`  - Total number of bytes from offset `i` to be read from `b`. 
    </ul>
   </dd>
</dl>


-------------------------------------------------------------------------------
#### bytes.get_byte()

 Get the byte at an  offset `i` from Bytes `b`.

```lua
function bytes.get_byte(b:Bytes, i: Integer): Byte
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to get the bytes from. 
      <li>`i`  - Offset from which the bytes should be read. 
    </ul>
   </dd>
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int16(), bytes.get_int16_be()

Get a big endian 16-bit integer at specified offset.  get_int16_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.get_int16_be(b: Bytes, offset:  Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 16-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int16_le()

Get a little endian 16-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.get_int16_le(b: Bytes, offset:  Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 16-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int32(), bytes.get_int32_be()

Get a big endian 32-bit integer at specified offset.  get_int32_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.get_int32_be(b: Bytes, offset: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 32-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int32_le()

Get a little endian 32-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.get_int32_le(b: Bytes, offset: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 32-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int64(), bytes.get_int64_be()

Get a big endian 64-bit integer at specified offset.  get_int64_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.get_int64_be(b: Bytes, offset: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 64-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_int64_le()

Get a little endian 64-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.get_int64_le(b: Bytes, offset: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The 64-bit integer value at the specified offset.
</dl>


-------------------------------------------------------------------------------
#### bytes.get_var_int()

Get an integer that was encoded in 7-bit format at given offset.  The high bit of each byte indicates if the next byte should be read.  Available since Aerospike Server 3.3.14.

```lua
function bytes.get_var_int(b: Bytes, offset: Integer): Integer, Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
    </ul>
  </dd>
  <dt>Returns
  <dd>The integer value at the specified offset and the encoded byte size of the integer.
</dl>

```lua
> local b = bytes(32)
> -- Write string with 7-bit encoded length.
> local size = bytes.set_var_int(b, 1, 8)
> bytes.set_string(b, size + 1, 'mystring')
> -- Read string.
> local length, size = bytes.get_var_int(b, 1)
> bytes.get_string(b, size + 1, length)
mystring
```


-------------------------------------------------------------------------------
#### bytes.set_string()

Set at offset of Bytes `b` a String `s`.

```lua
function bytes.set_string(b: Bytes, offset: Integer, s: String): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`s` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_bytes()

Set bytes at offset of Bytes `b`.

```lua
function bytes.set_bytes(b: Bytes, offset: Integer, source: Bytes, length: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the target bytes.
      <li>`offset` – the target bytes offset.
      <li>`source` – the source bytes.
      <li>`length` – the number of bytes to store.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_byte()

Set at offset of Bytes `b` a byte value `v`.

```lua
function bytes.set_byte(b: Bytes, offset: Integer, v: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`v` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int16(), bytes.set_int16_be()

Set big endian 16-bit integer at specified offset.  set_int16_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.set_int16_be(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int16_le()

Set little endian 16-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.set_int16_le(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int32(), bytes.set_int32_be()

Set big endian 32-bit integer at specified offset.  set_int32_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.set_int32_be(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int32_le()

Set little endian 32-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.set_int32_le(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int64(), bytes.set_int64_be()

Set big endian 64-bit integer at specified offset.  set_int64_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.set_int64_be(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_int64_le()

Set little endian 64-bit integer at specified offset.  Available since Aerospike Server 3.3.14.

```lua
function bytes.set_int64_le(b: Bytes, offset: Integer, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.set_var_int()

Encode an integer into 7-bit format at given offset.  The high bit of each byte indicates if the next byte should be read.  Available since Aerospike Server 3.3.14.

```lua
function bytes.set_var_int(b: Bytes, offset: Integer, i: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`offset` – the offset of the value.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>The encoded byte size of the integer.
</dl>

```lua
> local b = bytes(32)
> -- Write string with 7-bit encoded length.
> local size = bytes.set_var_int(b, 1, 8)
> bytes.set_string(b, size + 1, 'mystring')
> -- Read string.
> local length, size = bytes.get_var_int(b, 1)
> bytes.get_string(b, size + 1, length)
mystring
```


-------------------------------------------------------------------------------
#### bytes.append_string()

Append a NULL-terminated string value to `Bytes` `b`.

```lua
function bytes.append_string(b:Bytes, v: String): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to set the value in. 
      <li>`v`  - The NULL-terminated string value to append to `b`.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>

-------------------------------------------------------------------------------
#### bytes.append_bytes()

Append `Bytes` `v` to `Bytes` `b`.

```lua
function bytes.append_bytes(b:Bytes, v:Bytes, n:Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to set the value in. 
      <li>`v`  - The `Bytes` value to append to `b`.
      <li>`n`  - The number of bytes to append to `b`.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_byte()

Append a byte to `Bytes` `b`.

```lua
function bytes.append_byte(b:Bytes, v:Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b`  - The `Bytes` to set the value in. 
      <li>`v`  - The byte value to append to `b`.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int16(), bytes.append_int16_be()

Append big endian 16-bit integer to `Bytes` `b`.  append_int16_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.append_int16_be(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int16_le()

Append little endian 16-bit integer to `Bytes` `b`.  Available since Aerospike Server 3.3.14.

```lua
function bytes.append_int16_le(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int32(), bytes.append_int32_be()

Append big endian 32-bit integer to `Bytes` `b`.  append_int32_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.append_int32_be(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int32_le()

Append little endian 32-bit integer to `Bytes` `b`.  Available since Aerospike Server 3.3.14.

```lua
function bytes.append_int32_le(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int64(), bytes.append_int64_be()

Append a big endian 64-bit integer value to `Bytes` `b`.  append_int64_be() is available since Aerospike Server 3.3.14.

```lua
function bytes.append_int64_be(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_int64_le()

Append a little endian 64-bit integer value to `Bytes` `b`.  Available since Aerospike Server 3.3.14.

```lua
function bytes.append_int64_le(b: Bytes, i: Integer): Boolean
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to set the value of.
      <li>`i` – The integer value to append to b.
    </ul>
  </dd>
  <dt>Returns
  <dd>If command succeeded.
</dl>


-------------------------------------------------------------------------------
#### bytes.append_var_int()

Append an integer into 7-bit format.  The high bit of each byte indicates if the next byte should be read.  Available since Aerospike Server 3.3.14.

```lua
function bytes.append_var_int(b: Bytes, i: Integer): Integer
```

<dl class="function_spec">
  <dt>Parameters
  <dd>
    <ul>
      <li>`b` – the `Bytes` to get the value of.
      <li>`i` – the value to be stored at the offset.
    </ul>
  </dd>
  <dt>Returns
  <dd>The encoded byte size of the integer.
</dl>

```lua
> local b = bytes(32)
> -- Append string with 7-bit encoded length.
> local size = bytes.append_var_int(b, 8)
> bytes.append_string(b, 'mystring')
> -- Read string.
> local length, size = bytes.get_var_int(b, 1)
> bytes.get_string(b, size + 1, length)
mystring
```

