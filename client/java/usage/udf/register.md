---
title: Register a UDF
description: Register a module containing a UDF asynchronously with the Aerospike server.
---

The Aerospike Java client can be used to register user defined functions (UDFs).  UDFs are written in the [Lua](https://www.lua.org) programming language that execute in the Aerospike server process.  These functions can be grouped together into [Lua packages](https://www.lua.org/pil/15.html) and registered with the Aerospike server cluster. Here is an example Lua package:

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
* ASCII Text File Embedded in a Java Resource
* Java String

### Register UDF in ASCII Text File

```java
RegisterTask task = client.register(null, "/home/user/udf/example.lua", "example.lua", Language.LUA);
// Poll cluster for completion every second for a maximum of 10 seconds.
task.waitTillComplete(1000, 10000);
```

The UDF Lua package will be registered on all server's in the cluster.  `/home/user/udf/example.lua` is location of the Lua package on the client's filesystem.  `example.lua` is the filename of the Lua package relative to each server's configured mod-lua user-path in `aerospike.conf`.

```
mod-lua {
    user-path /opt/aerospike/usr/udf/lua
}
```

In this configuration, the Lua package's full path on each server's filesystem will be `/opt/aerospike/usr/udf/lua/example.lua`.

The registration is asynchronous on the server.  The server will immediately return an acknowledgement of the registration command and then register the UDF with other servers in the cluster.  The client has the option of waiting for the registration task to complete using the returned RegisterTask instance.


### Register UDF in Java Resource

The UDF Lua package text file can be embedded into in a [Java resource](https://maven.apache.org/guides/getting-started/index.html#How_do_I_add_resources_to_my_JAR) in the user's application jar.  In this case, the UDF Lua package is placed in the
user's java source code tree `src/main/resources/udf/example.lua`.  The resource file will be included in the user's application jar on compilation.  Registration is performed by:

```java
RegisterTask task = client.register(null, MyClass.class.getClassLoader(), "udf/example.lua", "example.lua", Language.LUA);
// Poll cluster for completion every second for a maximum of 10 seconds.
task.waitTillComplete(1000, 10000);
```

In order to read the resource, the classloader from a user's application class (MyClass) must be provided. 
`udf/record_example.lua` is the relative path of the resource from the root resource dir.  `example.lua` is the filename of the Lua package relative to each server's configured mod-lua user-path in `aerospike.conf`.


### Register UDF in Java String

```java
String newline = "\n";
String code = 
	"-- Validate value before writing." + newline + 
	"function writeWithValidation(r,name,value)" + newline + 
	"    if (value >= 1 and value <= 10) then" + newline + 
	"      if not aerospike:exists(r) then" + newline + 
	"        aerospike:create(r)" + newline + 
	"      end" + newline + 
	"      r[name] = value" + newline + 
	"      aerospike:update(r)" + newline + 
	"    else" + newline + 
	"        error("1000:Invalid value")" + newline +  
	"    end" + newline + 
	"end" + newline + 
	newline + 
	"-- Set a particular bin only if record does not already exist." + newline +
	"function writeUnique(r,name,value)" + newline +
	"    if not aerospike:exists(r) then" + newline +
	"        aerospike:create(r)" + newline +
	"        r[name] = value" + newline +
	"        aerospike:update(r)" + newline +
	"    end" + newline +
	"end" + newline;
   
RegisterTask task = client.registerUdfString(null, code, "example.lua", Language.LUA);
// Poll cluster for completion every second for a maximum of 10 seconds.
task.waitTillComplete(1000, 10000);
```

`example.lua` is the filename of the Lua package relative to each server's configured mod-lua user-path in `aerospike.conf`.
