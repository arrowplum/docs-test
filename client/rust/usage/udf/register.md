---
title: Register a UDF
description: Register a module containing a UDF asynchronously with the Aerospike server.
---

The Aerospike Rusts client can be used to register user-defined functions
(UDFs). UDFs are written in the [Lua](https://www.lua.org) programming
language that execute in the Aerospike server process. These functions can be
grouped together into [Lua packages](https://www.lua.org/pil/15.html) and
registered with the Aerospike server cluster. Here is an example Lua package:

```lua
-- Validate value before writing.
function writeWithValidation(r,name,value)
    if (value >= 1 and value <= 10) then
      if not aerospike:exists(r) then 
        aerospike:create(r)
      end
      r[name] = value
      aerospike:update(r)
    else
        error("1000:Invalid value") 
    end
end

-- Set a particular bin only if record does not already exist.
function writeUnique(r,name,value)
    if not aerospike:exists(r) then 
        aerospike:create(r) 
        r[name] = value
        aerospike:update(r)
    end
end
```

The UDF Lua package can be created in one of the following formats:

* ASCII Text File
* String

### Register UDF in ASCII Text File

```rust
client.register_udf(&WritePolicy::default(), "/home/user/udf/example.lua", "example.lua", UDFLang::Lua).unwrap();
```

The UDF Lua package will be registered on all server's in the cluster.
`/home/user/udf/example.lua` is location of the Lua package on the client's
filesystem. `example.lua` is the filename of the Lua package relative to each
server's configured mod-lua user-path in `aerospike.conf`.

```
mod-lua {
    user-path /opt/aerospike/usr/udf/lua
}
```

In this configuration, the Lua package's full path on each server's filesystem
will be `/opt/aerospike/usr/udf/lua/example.lua`.

The registration is asynchronous on the server. The server will immediately
return an acknowledgement of the registration command and then register the UDF
with other servers in the cluster. The client has the option of waiting for the
registration task to complete using the returned RegisterTask instance.

### Register UDF in a String

```rust
let code = r#"
-- Validate value before writing.
function writeWithValidation(r,name,value)
    if (value >= 1 and value <= 10) then
      if not aerospike:exists(r) then
        aerospike:create(r)
      end
      r[name] = value
      aerospike:update(r)
    else
        error("1000:Invalid value")
    end
end

-- Set a particular bin only if record does not already exist.
function writeUnique(r,name,value)
    if not aerospike:exists(r) then
        aerospike:create(r)
        r[name] = value
        aerospike:update(r)
    end
end
"#;

client.register_udf(&WritePolicy::default(), code.as_bytes(), "example.lua", UDFLang::Lua).unwrap();
```

`example.lua` is the filename of the Lua package relative to each server's
configured mod-lua user-path in `aerospike.conf`.
