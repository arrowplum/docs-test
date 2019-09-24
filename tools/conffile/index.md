---
title: Aerospike Tools Configuration File
description: Learn the details of Aerospike Tools Configuration File and its format.
---

Aerospike tools can read startup options from configuration files. Configuration files are an easy way to specify commonly used options and also manage multiple options in groups.

Invoke a tool with the --help option to see if it has the capability to read a configuration file.

A tool which supports a configuration file has a help message indicating the allowable options and the group that it belongs to.

Example:

```asciidoc
 --no-config-file
                      Do not read any config file. Default: disabled
 --instance <name>
                      Section with these instance is read. e.g in case instance `a` is specified
                      sections cluster_a, aql_a is read.
 --config-file <path>
                      Read this file after default configuration file.
 --only-config-file <path>
                      Read only this configuration file.
```

### Defaults
On Unix and Unix-like systems, the configuration file options for the Aerospike tools is shown in the following table. The precedence of the files is from top to bottom (top files are read first, files read later take precedence).

Following are default configuration files read in the order:

File Name |	Purpose
---|---
/etc/aerospike/astools.conf | Global options
~/.aerospike/astools.conf | User specific options
config-file |	The file specified with --config-file, if any. 

In the preceding table, ~ represents the current user's home directory (the value of $HOME).

{{#info}}
**Note:** The user can override one or more values read from the configuration file by specifying a value on the command line.
{{/info}}


The following options affect configuration file reading behavior:

* *--no-config-file*: does not load any configuration file. 
* *--config-file &lt;path&gt;*: file to be read after loading the default configuration file.
* *--only-config-file &lt;path&gt;*: skips default configuration files and only reads the user specified configuration file.

{{#info}}
**Note:** For using the config file feature with asadm and asinfo, additional dependencies are required: toml and jsonschema. To workaround those dependencies, use the --no-config-file option.
{{/info}}

The following description how options are specified while invoking tools.

### Command line syntax

option type | Usage | contraints
---|---|---
Long options | Use "equals":    --option=value. |No spaces.  No concat.
Short option -P | Use "concat":    -Pvalue. | No spaces.  No equals.
Short option for all others | Only support "space" or "concat":    -O value. |    No equals.

There are some special cases. For password an empty string can be provided in order to trigger a prompt. And in case the value in the options has a preceding `-` (hyphen) an extra space must be left, e.g --tls-protocol=" -all, +TLSv1.2"

### Configuration file Format

The Aerospike tools configuration file is a plain text file and can be created using any text editor. It is in TOML file format with limited support. 

Any long option that may be given on the command line when running any aerospike tools can be given in a configuration file as well. To get the list of available options for a program, run it with the --help option. For example, asadm --help.

The syntax for specifying options in a configuration file is similar to command-line syntax. However, in a configuration file, the leading two dashes are omitted from the option name and only one option per line is specified. For example, --port=3000 and --host=localhost on the command line should be specified as port=3000 and host="localhost" on separate lines in a configuration file.

Empty lines in the configuration files are ignored. Non empty lines can take any of the following forms:

```asciidoc
# comment

Comment lines start with # or ;. A # comment can start in the middle of a line as well.

[group]

group is the name of the program or group for which you want to set options. 
After a group line, any option-setting lines apply to the named group until 
the end of the configuration file or another group line is given. 
Option group names are not case-sensitive.

opt_name

This is equivalent to --opt_name on the command line.

opt_name=value

This is equivalent to --opt_name=value on the command line. 
In a configuration file, you can have spaces around the = character, 
something that is not true on the command line. The value optionally 
can be enclosed within single quotation marks or double quotation 
marks, which is useful if the value contains a # comment character. 
```

Leading and trailing spaces are automatically deleted from option names and values.

Following rules apply to the options and value in the configuration file:

1. Option values are of string, integer and boolean type. String are to be specified in quotes. Any other value type would fail with an error message. 

Some examples:

```bash
host="localhost:tls-name:3000"
port=3000
tls-enable=true
```

2. Multiple instances of the same `option` or `group` in the same file is not allowed.

3. In case of multiple instances of a given `option` or `group` in different file, the last instance takes precedence.

If an option group name is the same as a program name, options in the group apply specifically to that program. For example, the `[asadm]` and `[aql]` groups apply to the asadm and the aql program, respectively.

The `[cluster]` group is read by all tools, under which the connection end point, security and encryption options are specified.

Here is a typical global configuration file:

```asciidoc
[cluster]
host="localhost:3000"
port=3000
tls-enable=false
user="" # Empty string is for no user.

[asadm]
service-alumni=false

[aql]
outputmode="table"
```

### Includes

It is possible to use [include] group in configuration files to include other configuration files and to search specific directories for configuration files. For example, to include the /home/mydir/astools.conf file and to search the /home/mydir directory and read configuration files found there, use this directive:

[include] 
file="/home/mydir/astools.conf"
directory="/home/mydir"

Aerospike makes no guarantee about the order in which configuration files in the directory will be read. 

Options are read in following order:
1. Configuration file which has an include group.
2. Include files.
3. Include directory.

The contents of an included configuration file are like any other configuration file. That is, it should contain groups of options, each preceded by a [group] line that indicates the program to which the options apply.


```asciidoc
  [cluster]
  host="localhost"
  
  [cluster_red]
  host="redhost"
  
  [asadm]
  timeout=5
  
  [asadm_red]
  timeout=1000
```

### Instances

A group in the configuration file can be suffixed with the instance name. This allows the capability to manage groups using the name.

The group name without an instance suffix is refered to as global. By default, global groups are loaded. While invoking tools, the command line option --instance can be used to load a specific instance of a group. 

For example:

asadm --instance red

Like any other group, if multiple copies of the same instance is found, the later copy takes precedence. 

If the user invokes a tool with asadm --instance red:

1. Global configuration is ignored when a specific instance is specified, i.e. in case both [cluster] and [cluster_red] is specified, then the value in group [cluster] is ignored. 

2. In the case where [cluster\_red] is not specified, startup would fail with an error. To start a program with a certain instance, [cluster\_&lt;instancename&gt;] is compulsory.
 
3. In the case of a program specific group, the default is global. i.e. in case [asadm_red] is not found, options are read from the [asadm] group. 

4. The [include] group does not allow for a suffix, it is only global. 
