---
title: ascli – UDF Management
description: Learn about the ascli put, list, get and record-apply commands for managing user-defined functions.

breadcrumbs:
  - title: Aerospike 3.0
    url: /docs/v3
  - title: Tools and Utilities
    url: /docs/v3/Tools and Utilities.html
  - title: Aerospike CLI (ascli)
    url: /docs/v3/ascli.html
---

{{#note}}
Deprecated from Aerospike Tools Versions >= 3.9.0. Use aql instead.
{{/note}}

The following are ascli commands for managing user-defined functions.


### udf-put

To upload a UDF module and register the package, use the udf-put command:

```bash
ascli udf-put <filepath>
```

You can specify the path to the file. The file itself will be uploaded, but the directories will not be created.

Example: 

```bash
$ ascli udf-put example1.lua
```

Where example1.lua is a file containing:

```bash
function hello()
  return "Hello World!"
end
```

### udf-list

To list UDF modules, use the udf-list command:

```bash
ascli udf-list
```

Example:

```bash
$ ascli udf-list
example1.lua
```

### udf-get

To download the contents of a UDF module, use the udf-get command:

```bash
ascli udf-get <filename>
```

This will output the contents of the file.

Example:

```bash
$ ascli udf-get example1.lua
function hello()
  return "Hello World!"
end
```

# udf-remove

To remove a UDF module, use the udf-remove command:

```bash
ascli udf-remove <filename>
```

Example:

```bash
$ ascli udf-remove example1.lua
```

### udf-record-apply

To call a function in a UDF file, use the udf-record-apply command:

```bash
ascli udf-record-apply <ns> <set> <key> <module> <function> ARGS
```

The `<ns>` parameter is the namespace for the record to apply the UDF to.

The `<set>` parameter is the set for the record to apply the UDF to.

The `<key>` parameter is the key for the record to apply the UDF to.

The `<module>` parameter is the UDF module name containing the function to be applied to the record. This is the base name of the module file, without the extension.

The `<function>` parameter is the name of the UDF to be applied to the record.

The ARGS are the arguments to be passed to the UDF.

Example:

```bash
$ ascli udf-record-apply test test 1 example1 hello
"Hello World!"
```
