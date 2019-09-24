---
title: Aerospike Admin User Guide
description: Introduction to Aerospike Admin commands and common patterns.

---

### Install
To install Aerospike Admin either install the **Aerospike-Tools** package
which is bundled in our [Server Packages](/download/) or clone and install from
the [Github Repository](https://github.com/aerospike/aerospike-admin).

### Getting Help
1. Run `asadm --help` to learn about arguments that may be needed for your
environment.
2. To start the interactive shell run `asadm -h [HOST_IP]`.
   * From 0.1.5 onwards `asadm` supports IPv6 address for server 3.10 and above.
3. To start the interactive shell in Collectinfo-analyzer mode run `asadm -c -f <collectinfopath>` (**Introduced:** 0.1.8).
   * -f accepts path of single collectinfo file or collectinfo directory path.
4. To start the interactive shell in Log-analyzer mode run `asadm -l`.
   * -f parameter is used to set aerospike log. Also we can add server logs at runtime.
5. From the `Admin>` prompt run `help` to see a complete list of supported
   commands.
6. Non-interactive execution (**Introduced:** 0.1.1):
   * To execute inline command/s run `asadm -e [; separated commands]`. Please use `-c` for Collectinfo-analyzer and `-l` for Log-analyzer commands.
   * To execute command/s from file run `asadm -e [COMMAND_FILE_PATH]`. Please use `-c` for Collectinfo-analyzer and `-l` for Log-analyzer commands.
	* Please see [Command File Format](#command-file-format) for details.
   * To get output in file run `asadm -e [COMMANDS] -o [OUTPUT_FILE_PATH]`

### Command Organization
Commands are organized into a hierarchy where similar commands share a common
parent. Each command may support one or more **modifiers** which can limit the
command to a specific set of nodes or filter the output of a command.

All commands will sort output based on the **node name** that asadm chooses for
the node. The **node name** is Fully Qualified Domain Name (FQDN) or IP if the 
FQDN cannot be resolved.

### Command Shortcuts
All commands support shortest prefix execution and tab completion; at the
**Admin>** prompt, typing `i<tab>` will complete to `info` and `sh<tab>` will
suggest `shell` and `show`. You can execute commands using their shortest
unambiguous prefix for example to run `info` you could type `i` and for
`info network` you could type `i net`, while `i n` would be ambiguous with
`info namespace` and `info network`.

```asciidoc
Admin> i net
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Network Information (2018-03-02 08:28:09 UTC)~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
             Node               Node                  Ip        Build   Cluster   Migrations        Cluster     Cluster         Principal   Client     Uptime   
                .                 Id                   .            .      Size            .            Key   Integrity                 .    Conns          .   
172.28.128.3:3000   *BB9635C2A270008   172.28.128.3:3000   E-3.15.0.2         1      0.000     1CEC1DC68B04   True        BB9635C2A270008        5   01:23:56   
Number of rows: 1

Admin> i n
ERR: Ambiguous command: 'n' may be namespace or network.
```

### Modifiers
There are currently four modifiers that commands may support.
* **diff**: Diff shows the difference between the configurations in all nodes of cluster.
    * Only `show config` command supports `diff` modifier
    * Example: Show difference in service configurations
    ```asciidoc
    show config service diff
    ```
* **like**: Like shows the output containing any word listed after 'like'.
    * Example: Show namespace stats which has substring object
    ```asciidoc
    show statistics namespace like object
    ```
* **with**: With specifies a list of nodes to be used with the given command. It works only in Cluster mode.
    * You can use the node's IP, FQDN, or Node ID as well as any unique prefix.
    * Example: Show XDR configuration for node with substring 172.28
    ```asciidoc
    show config xdr like object with 172.28
    ```
* **for**: For shows the output for specific namespaces matching pattern listed after ‘for’
    * `show statistics namespace`, `show statistics sets`, `show statistics bins`, `show statistics sindex`, `show config namespace` and `show distribution` commands support `for` modifier
        * Example: Show distribution for namespace has substring test
        ```asciidoc
        show distribution for test
        ```
    * `show latency` command supports `for` modifier if server version is 3.9 and above
        * Example: Show latency for namespace has substring test or bar
        ```asciidoc
        show distribution for “test|bar”
        ```
    * `show statistics set` can accept two patterns after ‘for’, first for namespace and second for set
        * Example: Show statistics for set has substring ABCD or ends with XYZ for namespace starts with test
        ```asciidoc
        show statistics set for "^test.*" "ABCD|.*XYZ$"
        ```
    * `show statistics sindex` can accept two patterns after ‘for’, first for namespace and second for sindex
        * Example: Show statistics for sindex named "ABCD" for namespace test
        ```asciidoc
        show statistics sindex for test "^ABCD$"
        ```
	
### Modes
There are currently three modes asadm supports.
* **cluster**: This is default mode for asadm. This is used to get summary info for the current health of a running cluster and also for executing dynamic configuration and tuning commands across the running cluster. For a tutorial on commands to use in this mode see the [Cluster Mode Commands](/docs/tools/asadm/user_guide/cluster_mode_guide).
* **Collectinfo-analyzer**: In this mode, asadm works on collectinfo instead of running cluster. We can enable this mode by setting `-c` parameter. For a tutorial on commands to use in this mode see the [Collectinfo-analyzer Mode Commands](/docs/tools/asadm/user_guide/collectinfo_analyser_mode_guide).
   * Compatibility: collectinfo file/folder created by asadm
* **Log-analyzer**: In this mode, asadm works on aerospike server log files instead of running cluster. We can enable this mode by setting `-l` parameter. For a tutorial on commands to use in this mode see the [Log-analyzer Mode Commands](/docs/tools/asadm/user_guide/log_analyser_mode_guide).

### Command File Format
We can execute multiple commands from file by using `-e` option. This command file should be as below:
* Command should end with `;`.
* Command can be splitted on multiple lines.
* File should have commands for single mode.
* For single line comment use `//`
* For multiple line comment use  `/* … */`. `/*` should be start of line and `*/` should be end of line.

Example:
```asciidoc
show config;
info namespace;
// This is single line comment
show 
 statistics;
/* This is multiple line comment - line 1
This is multiple line comment - line 2 */
show statistics like obj;
```

