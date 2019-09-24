---
title: Aerospike Admin (asadm)
description: Learn about the Aerospike Admin tool, asadm, the place to start when monitoring performance, tuning configuration settings and diagnosing issues with your cluster.

---

Aerospike Admin is an interactive python utility primarily used to get
summary info for the current health of a cluster and also for executing dynamic
configuration and tuning commands across the cluster.

Aerospike Admin is provided by the **Aerospike-Tools** package which is bundled
within our various [Server Packages](/download). Optionally you may obtain it
through the
[Aerospike-Admin repository](http://www.github.com/aerospike/aerospike-admin).

Refer to `asadm --help` below for a list of parameters that may be required to
run asadm within your cluster. asadm can be configured by using tools configuration files. 
Please see [Aerospike Tools Configuration](/docs/tools/conffile) for more details.

```bash
asadm --help

Usage: asadm [OPTIONS]
---------------------------------------------------------------------------------------

 -V --version         Show the version of asadm and exit
 -E --help            Show program usage.
 -e --execute         Execute a single or multiple asadm commands and exit. 
                      The input value is either string of ';' separated asadm 
                      commands or path of file which has asadm commands (ends with ';').
 -o --out-file        Path of file to write output of -e command[s].
 --no-color           Disable colored output.
 --single-node        Enable asadm mode to connect only seed node. 
                      By default asadm connects to all nodes in cluster.
 --collectinfo        Start asadm to run against offline collectinfo files.
 --log-analyser       Start asadm in log-analyser mode and analyse data from log files.
 -f --log-path        Path of cluster collectinfo file or directory 
                      containing collectinfo and system info files.


Configuration File Allowed Options
----------------------------------

[cluster]
 -h HOST, --host=HOST
                      HOST is "<host1>[:<tlsname1>][:<port1>],..." 
                      Server seed hostnames or IP addresses. The tlsname is 
                      only used when connecting with a secure TLS enabled 
                      server. Default: localhost:3000
                      Examples:
                        host1
                        host1:3000,host2:3000
                        192.168.1.10:cert1:3000,192.168.1.20:cert2:3000
 -p PORT, --port=PORT 
                      Server default port. Default: 3000
 -U USER, --user=USER 
                      User name used to authenticate with cluster. Default: none
 -P, --password
                      Password used to authenticate with cluster. Default: none
                      User will be prompted on command line if -P specified and no
                      password is given.
 --tls-enable         Enable TLS on connections. By default TLS is disabled.
 --tls-cafile=TLS_CAFILE <path>
                      Path to a trusted CA certificate file.
 --tls-capath=TLS_CAPATH <path>
                      Path to a directory of trusted CA certificates.
 --tls-protocols=TLS_PROTOCOLS
                      Set the TLS protocol selection criteria. This format
                      is the same as Apache's SSLProtocol documented at http
                      s://httpd.apache.org/docs/current/mod/mod_ssl.html#ssl
                      protocol . If not specified the asadm will use '-all
                      +TLSv1.2' if has support for TLSv1.2,otherwise it will
                      be '-all +TLSv1'.
 --tls-cipher-suite=TLS_CIPHER_SUITE
                      Set the TLS cipher selection criteria. The format is
                      the same as Open_sSL's Cipher List Format documented
                      at https://www.openssl.org/docs/man1.0.1/apps/ciphers.
                      html
 --tls-keyfile=TLS_KEYFILE <path>
                      Path to the key for mutual authentication (if
                      Aerospike Cluster is supporting it).
 --tls-keyfile-password=password
                      Password to load protected tls-keyfile.
                      It can be one of the following:
                      1) Environment varaible: 'env:<VAR>'
                      2) File: 'file:<PATH>'
                      3) String: 'PASSWORD'
                      Default: none
                      User will be prompted on command line if --tls-keyfile-password specified and no
                      password is given.
 --tls-certfile=TLS_CERTFILE <path>
                      Path to the chain file for mutual authentication (if
                      Aerospike Cluster is supporting it).
 --tls-cert-blacklist <path>
                      Path to a certificate blacklist file. The file should
                      contain one line for each blacklisted certificate.
                      Each line starts with the certificate serial number
                      expressed in hex. Each entry may optionally specify
                      the issuer name of the certificate (serial numbers are
                      only required to be unique per issuer).Example:
                      867EC87482B2
                      /C=US/ST=CA/O=Acme/OU=Engineering/CN=TestChainCA
 --tls-crl-check      Enable CRL checking for leaf certificate. An error
                      occurs if a valid CRL files cannot be found in
                      tls_capath.
 --tls-crl-check-all  Enable CRL checking for entire certificate chain. An
                      error occurs if a valid CRL files cannot be found in
                      tls_capath.

[asadm]
 -t --tls-name        Default TLS name of host to verify for TLS connection,
                      if not specified in host string. It is required if tls-enable
                      is set.
 -s --services-alumni
                      Enable use of services-alumni-list instead of services-list
 -a --services-alternate 
                      Enable use of services-alternate instead of services in
                      info request during cluster tending
 --timeout            Set timeout value in seconds to node level operations. 
                      TLS connection does not support timeout. Default: 5 seconds



Default configuration files are read from the following files in the given order:
/etc/aerospike/astools.conf ~/.aerospike/astools.conf
The following sections are read: (cluster asadm include)
The following options effect configuration file behavior

 --no-config-file
                      Do not read any config file. Default: disabled
 --instance <name>
                      Section with these instance is read. e.g in case instance 
                      `a` is specified sections cluster_a, asadm_a is read.
 --config-file <path>
                      Read this file after default configuration file.
 --only-config-file <path>
                      Read only this configuration file.
```

After executing **asadm** you will receive an 'Admin>' prompt, type help
and press return for a list of commands and descriptions.

```asciidoc
Aerospike Admin
  - asinfo:
    "asinfo" provides raw access to the info protocol.
      Options:
        -v <command>   - The command to execute
        -p <port>      - Port to use in case of XDR info command
                         and XDR is not in asd
        -l             - Replace semicolons ";" with newlines.
        --no_node_name - Force to display output without printing node names.
    Modifiers: like, with
    Default: Executes an info command.
  - collectinfo:
    "collectinfo" is used to collect cluster info, aerospike conf file and system stats.
    Modifiers: with
    Default: Collects cluster info, aerospike conf file for local node and system stats from all nodes if remote server credentials provided.
    If credentials are not available then it will collect system stats from local node only.
      Options:
        -n           <int>           - Number of snapshots. Default: 1
        -s           <int>           - Sleep time in seconds between each snapshot. Default: 5 sec
        --enable-ssh                 - Enable remote server system statistics collection.
        --ssh-user   <string>        - Default user id for remote servers. This is System user id (not Aerospike user id).
        --ssh-pwd    <string>        - Default password or passphrase for key for remote servers. This is System password (not Aerospike password).
        --ssh-port   <int>           - Default SSH port for remote servers. Default: 22
        --ssh-key    <string>        - Default SSH key (file path) for remote servers.
        --ssh-cf     <string>        - Remote System Credentials file path.
                                       If server credentials are not available in credential file then default credentials will be used 
                                       File format : each line should contain <IP[:PORT]>,<USER_ID>,<PASSWORD or PASSPHRASE>,<SSH_KEY>
                                       Example:  1.2.3.4,uid,pwd
                                                 1.2.3.4:3232,uid,pwd
                                                 1.2.3.4:3232,uid,,key_path
                                                 1.2.3.4:3232,uid,passphrase,key_path
                                                 [2001::1234:10],uid,pwd
                                                 [2001::1234:10]:3232,uid,,key_path
        --output-prefix <string>     - Output directory name prefix.
        --asconfig-file <string>     - Aerospike config file path to collect. Default: /etc/aerospike/aerospike.conf
  - exit:
    Terminate session
  - features:
    Displays features used in running Aerospike cluster.
    Modifiers: like, with
  - help:
    Returns documentation related to a command
    for example, to retrieve documentation for the "info"
    command use "help info".
  - info:
    The "info" command provides summary tables for various aspects
    of Aerospike functionality.
    Modifiers: with
    Default: Displays network, namespace, and XDR summary information.
      - dc:
        Displays summary information for each datacenter.
      - namespace:
        The "namespace" command provides summary tables for various aspects
        of Aerospike namespaces.
        Modifiers: with
        Default: Displays usage and objects information for namespaces
          - object:
            Displays object information for each namespace.
          - usage:
            Displays usage information for each namespace.
      - network:
        Displays network information for Aerospike.
      - set:
        Displays summary information for each set.
      - sindex:
        Displays summary information for Secondary Indexes (SIndex).
      - xdr:
        Displays summary information for Cross Datacenter
        Replication (XDR).
  - pager:
    Set pager for output
      - off:
        Removes pager and prints output normally
      - on:
        Displays output with vertical and horizontal paging for each output table same as linux 'less' command.
        Use arrow keys to scroll output and 'q' to end page for table.
        All linux less commands can work in this pager option.
      - scroll:
        Display output in scrolling mode
  - show:
    "show" is used to display Aerospike Statistics configuration.
      - config:
        "show config" is used to display Aerospike configuration settings
        Modifiers: diff, like, with
        Default: Displays service, network, and namespace configuration
          Options:
            -r <int>     - Repeating output table title and row header after every r columns.
                           default: 0, no repetition.
            -flip        - Flip output table to show Nodes on Y axis and config on X axis.
          - cluster:
            Displays Cluster configuration
          - dc:
            Displays datacenter configuration
          - namespace:
            Displays namespace configuration
          - network:
            Displays network configuration
          - service:
            Displays service configuration
          - xdr:
            Displays XDR configuration
      - distribution:
        "distribution" is used to show the distribution of object sizes
        and time to live for node and a namespace.
        Modifiers: for, with
        Default: Shows the distributions of Time to Live and Object Size
          - eviction:
            Shows the distribution of namespace Eviction TTLs for server version 3.7.5 and below
          - object_size:
            Shows the distribution of Object sizes for namespaces
              Options:
                -b               - Force to show byte wise distribution of Object Sizes.
                                   Default is rblock wise distribution in percentage
                -k <buckets>     - Maximum number of buckets to show if -b is set.
                                   It distributes objects in same size k buckets and 
                                   display only buckets which has objects in it. Default is 5.
          - time_to_live:
            Shows the distribution of TTLs for namespaces
      - latency:
        Modifiers: for, like, with
        Default: Displays latency information for Aerospike cluster.
          Options:
            -f <int>     - Number of seconds (before now) to look back to.
                           default: Minimum to get last slice
            -d <int>     - Duration, the number of seconds from start to search.
                           default: everything to present
            -t <int>     - Interval in seconds to analyze.
                           default: 0, everything as one slice
            -m           - Set to display the output group by machine names.
      - mapping:
        "show mapping" is used to display Aerospike mapping from IP to Node_id and Node_id to IPs
        Modifiers: like
        Default: Displays mapping IPs to Node_id and Node_id to IPs
          - ip:
            Displays IP to Node_id mapping
          - node:
            Displays Node_id to IPs mapping
      - pmap:
        Displays partition map analysis of Aerospike cluster.
      - statistics:
        Displays statistics for Aerospike components.
        Modifiers: for, like, with
        Default: Displays bin, set, service, and namespace statistics
          Options:
            -t           - Set to show total column at the end. It contains node wise sum for statistics.
            -r <int>     - Repeating output table title and row header after every r columns.
                           default: 0, no repetition.
            -flip        - Flip output table to show Nodes on Y axis and stats on X axis.
          - bins:
            Displays bin statistics
          - dc:
            Displays datacenter statistics
          - namespace:
            Displays namespace statistics
          - service:
            Displays service statistics
          - sets:
            Displays set statistics
          - sindex:
            Displays sindex statistics
          - xdr:
            Displays XDR statistics
  - summary:
    Displays summary of Aerospike cluster.
      Options:
        -l                        - Enable to display namespace output in List view. Default: Table view
        --enable-ssh              - Enable remote server system statistics collection.
        --ssh-user   <string>     - Default user id for remote servers. This is System user id (not Aerospike user id).
        --ssh-pwd    <string>     - Default password or passphrase for key for remote servers. This is System password (not Aerospike password).
        --ssh-port   <int>        - Default SSH port for remote servers. Default: 22
        --ssh-key    <string>     - Default SSH key (file path) for remote servers.
        --ssh-cf     <string>     - Remote System Credentials file path.
                                    If server credentials are not available in credential file then default credentials will be used 
                                    File format : each line should contain <IP[:PORT]>,<USER_ID>,<PASSWORD or PASSPHRASE>,<SSH_KEY>
                                    Example:  1.2.3.4,uid,pwd
                                              1.2.3.4:3232,uid,pwd
                                              1.2.3.4:3232,uid,,key_path
                                              1.2.3.4:3232,uid,passphrase,key_path
                                              [2001::1234:10],uid,pwd
                                              [2001::1234:10]:3232,uid,,key_path
    Modifiers: with
  - watch:
    "watch" Runs a command for a specified pause and iterations.
    Usage: watch [pause] [iterations] [--no-diff] command]
       pause:      the duration between executions.
                   [default: 2 seconds]
       iterations: Number of iterations to execute command.
                   [default: until keyboard interrupt]
       --no-diff:  Do not do diff highlighting
    Example 1: Show "info network" 3 times with 1 second pause
               watch 1 3 info network
    Example 2: Show "info namespace" with 5 seconds pause until
               interrupted
               watch 5 info namespace

```

For a tutorial on how to use **Aerospike Admin** see the
[User Guide](/docs/tools/asadm/user_guide).
