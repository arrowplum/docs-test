---
title: Directory Structure
description: Handy information about the Aerospike directory structure that will help store tools, system files, and data files. 
---

Aerospike uses a number of directories to store tools, system files, and data
files. This page describes the directories in use, although they are typically
managed through Aerospike tools, should you wish to manually manage these files.

## `/opt/aerospike`

This directory is created and managed by the Aerospike packages, when
installed through Linux package management. It contains a number of
sub-directories, some created by the tools package, others created by the
server package and maintained during server run-time.

## Current working directory

When using Aerospike through [source compilation](https://github.com/aerospike/aerospike-server)
or the [Other Linux](/docs/operations/install/linux/other) package, the run time
directories noted below will be created in the current working directory instead
of in `/opt/aerospike`

## Tools directories

These directories are installed through the tools package, if in use, and are
normally modified by adding and removing that package.

- `lib` - the simple Aerospike python client used by management tools.
- `bin` - [tools] binaries, such as
  [aql](/docs/tools/aql), [asadm](/docs/tools/asadm), [asbackup/asrestore](/docs/tools/backup),
  and others.
- `doc` - tools documentation and licenses.
- `examples` - `aql` sample files, and an example for using C with Lua.
   - The `examples` directory has been removed in Aerospike Tools 3.15.0.3.
   - For examples please refer to the
     [AQL](https://www.aerospike.com/docs/tools/aql/index.html)
     documentation.

## Run time directories

- `data`

  This directory is created by the install package to allow a place for
  Aerospike data files - which persist in-memory data. In standard operational
  configurations, the file system is not recommended, and instead data is stored
  on raw devices. However, for developer installation the file system is often
  preferred.

- `smd`

  The System MetaData directory contains a catalog of data kept in a
  distributed, persistent fashion. The format for these files is JSON, for
  readability - it is not recommended to edit these files manually. Information
  about the cluster's indexes, and registered User-Defined Functions, and other
  cluster-wide information is stored here.
  
  - evict.smd
  
    This module was added in [4.5.1.5](https://www.aerospike.com/download/server/notes.html#4.5.1.5).
    It stores the eviction real-time clock for each namespace.
  
  - roster.smd
  
    This module only applies to [Enterprise](https://www.aerospike.com/products/product-matrix/)
    [`strong-consistency`](https://www.aerospike.com/docs/reference/configuration/#strong-consistency)
    users and was added in
    [4.0.0.1](https://www.aerospike.com/download/server/notes.html#4.0.0.1).
    It stores the roster configuration for a each strong-consistency namespace.
  
  - security.smd
  
    This module only applies to Enterprise users. Stores
    [user permission](https://www.aerospike.com/docs/guide/security/access-control.html)
    definitions.
  
  - sindex.smd
  
    This module stores [secondary index](https://www.aerospike.com/docs/architecture/secondary-index.html)
    definitions.
  
  - truncate.smd
  
    This module stores per set or namespace LUT (Last Update Time) cutoffs.
  
  - UDF.smd
  
    This module stores UDF (User Defined Function) definitions, the actual UDFs
    are stored in `opt/aerospike/usr` directory

- `sys`

  This directory holds UDFs (and potentially other static content) which are
  part of the server package. They are maintained by the package manager, and
  are tested with the installed version of the server.

  Do not modify this directory; it will not be re-created on restart.

- `usr`

  This directory holds UDFs (and potentially other dynamic content) which are
  registered by administrators via cluster management tools such as
  [aql](/docs/tools/aql).
  
  It isn't recommended to directly modify files these files in production.
  However, in a developer environment, to speed of the development cycle, you
  may disable [Lua caching](https://www.aerospike.com/docs/reference/configuration/#cache-enabled)
  and modify files in `usr/udf/lua` directly.
