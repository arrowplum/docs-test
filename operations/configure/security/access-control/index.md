---
title: Configuring Access Control
description: Configure Access Control
---

## Access Control

Aerospike Access Control allows user, role, and privilege creation and maintenance.

Refer to the [Access Control guide](/docs/guide/security/access-control.html) for basic feature concepts.

## Enabling Access Control

A stanza must be added to every server in the cluster to enable Access Control. Access Control is available
only in Enterprise Edition. This *security* section to the Aerospike configuration file (*aerospike.conf*):

```bash
security {
    enable-security true

    # Other security-related configuration...
}
```

## Client version requirements

To use the Authentication system described below, please be aware of the minimum client version levels required.
- Java 3.1.2
- C / C++ 3.1.16
- C# .NET 3.1.2
- Python 1.0.44
- PHP 3.4.1
- Go 1.3.0

## Permissions

A user role is a set of *privileges*. A privilege consists of permission and scope. Scope specifies whether to apply the permission globally, per namespace, or per namespace and set.

There are seven permission levels: 

| Permission Level/Role | Scope |
| ---------- | ---------- | 
| read <br> <br> <br> <br> <br>|- get record <br>- scan<br>- query<br>- get server configuration and statistics <br>- change user password  | 
| write (separate as of Aerospike 4.6) <br>|- put, append, prepend, add, touch, delete |
| read-write  <br> <br>|- all read user role privileges <br>- all write user role privileges |
| read-write-udf  <br> <br> <br>|- all read-write user role privileges <br>- execute user-defined functions (UDFs) <br>- execute queries using UDFs |
| data-admin  <br> <br> <br> <br> <br>|- create, drop index <br>- register and remove UDFs <br>- use the scan-query job monitoring system <br>- abort scans and queries <br>- change user password |
| sys-admin  <br> <br> <br> <br>|- all data-admin role privileges <br>- set dynamic server configuration variables <br>- enable specialized logging <br>- get server configuration and statistics |
| user-admin  <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br>|- create and drop users <br>- change any user password  <br>- grant roles to users  <br>- revoke user roles <br>- create and drop user roles <br>- grant user role privileges <br>- revoke role privileges <br>- set whitelists for roles (as of Aerospike 4.6)<br>- query all users and their roles  <br>- query all roles and their privileges  <br>- get server configuration and statistics |

{{#note}}
Global scope permissions are only available for data-admin, sys-admin, and user-admin roles.
{{/note}}

{{#note}}
**Note:** When issuing batch transactions and have set-level permissions enforced, the sendSetName policy must be set to true (default is false). For Java, you can find the usage in the [API reference](https://www.aerospike.com/apidocs/java/com/aerospike/client/policy/BatchPolicy.html#sendSetName). 
{{/note}}

### Aerospike Defined Roles

Aerospike provides defined roles corresponding with each permission level. Each Aerospike defined role has one privilege that consists of a permission and a global scope.

{{#note}}
Permissions and their corresponding Aerospike defined role names are the same. It is important to pay attention to their context.
{{/note}}

## Create Users and Custom Roles

Administrators can create and edit custom roles consisting of one or more privileges. Admins create users and grant them one or (optionally) more roles, and can edit user role collections. 

{{#note}}
In Aerospike Enterprise Edition revisions 3.7.0.1 or later, you can create a user without a role to allow exclusive cluster statistics read access.
{{/note}}

{{#note}}
In Aerospike Enterprise Edition revisions 4.6 and later, custom roles may also be assigned a whitelist, a comma-separated list of one or more IP addresses. Users with such roles are only allowed to connect to the server from a whitelisted address.
{{/note}}

This example aql session demonstrates user and role management. For a security command summary, type `help` in aql and select *User Administration*.

```sql
$ aql -Uadmin -P
Enter Password:
Aerospike Query
Copyright 2013 Aerospike. All rights reserved.
aql> # change the admin user's password
aql> set password newpasswd for admin
OK
aql> # create a superuser role with all privileges
aql> create role superuser privileges read-write-udf,sys-admin,user-admin
OK
aql> # list all roles
aql> show roles
+------------------+-----------------------------------------+
| role             | privileges                              |
+------------------+-----------------------------------------+
| "data-admin"     | "data-admin"                            |
| "read"           | "read"                                  |
| "read-write"     | "read-write"                            |
| "read-write-udf" | "read-write-udf"                        |
| "superuser"      | "user-admin, sys-admin, read-write-udf" |
| "sys-admin"      | "sys-admin"                             |
| "user-admin"     | "user-admin"                            |
+------------------+-----------------------------------------+
7 rows in set (0.000 secs)
OK
aql> # list all roles -- Aerospike server 4.6 or later
aql> show roles
+------------------+-----------------------------------------+-----------+
| role             | privileges                              | whitelist |
+------------------+-----------------------------------------+-----------+
| "data-admin"     | "data-admin"                            | ""        |
| "read"           | "read"                                  | ""        |
| "read-write"     | "read-write"                            | ""        |
| "read-write-udf" | "read-write-udf"                        | ""        |
| "superuser"      | "user-admin, sys-admin, read-write-udf" | ""        |
| "sys-admin"      | "sys-admin"                             | ""        |
| "user-admin"     | "user-admin"                            | ""        |
| "write"          | "write"                                 | ""        |
+------------------+------------------+----------------------------------+
8 rows in set (0.000 secs)

aql> # create a user with the superuser role
aql> create user superman password krypton role superuser
OK
aql> # list all users
aql> show users
+------------+--------------+
| user       | roles        |
+------------+--------------+
| "admin"    | "user-admin" |
| "superman" | "superuser"  |
+------------+--------------+
2 rows in set (0.001 secs)
OK
aql> # create a role with read-write-udf privileges on set "setA" in namespace "test"
aql> create role setA-user privileges read-write-udf.test.setA
OK
aql> show role setA-user
+-------------+----------------------------+
| role        | privileges                 |
+-------------+----------------------------+
| "setA-user" | "read-write-udf.test.setA" |
+-------------+----------------------------+
1 row in set (0.000 secs)
OK
aql> # add a whitelist to this role so that it must connect from a 127.xx.xx.xx address
aql> # (Aerospike server 4.6 or later)
aql> set whitelist "127.0.0.0/8" for setA-user
OK
aql> show role setA-user
+-------------+----------------------------+----------------+
| role        | privileges                 | whitelist      |
+-------------+----------------------------+----------------+
| "setA-user" | "read-write-udf.test.setA" | "127.0.0.0/8" |
+-------------+----------------------------+----------------+
1 row in set (0.001 secs)
OK
aql> # create a role with read-write-udf privileges on set "setB" in namespace "test"
aql> create role setB-user privileges read-write-udf.test.setB
OK
aql> # create a role with read-write-udf privileges on set "setC" in namespace "test"
aql> # that is only allowed to connect from a specific IP address (Aerospike server 4.6 or later)
aql> create role setC-user privileges read-write-udf.test.setB whitelist "127.23.45.67"
OK
aql> # remove the whitelist from this role (Aerospike server 4.6 or later)
aql> set whitelist "" for setC-user
OK
aql> # create a user with several roles
aql> create user fred password fredspwd roles user-admin,setA-user,setB-user
OK
aql> show user fred
+--------+------------------------------------+
| user   | roles                              |
+--------+------------------------------------+
| "fred" | "setA-user, setB-user, user-admin" |
+--------+------------------------------------+
1 row in set (0.001 secs)
OK
aql> # create a user without a role
aql> create user sally password foo
OK
aql> show user sally
+-------+-------+
| user  | roles |
+-------+-------+
| sally | ""    |
+-------+-------+
aql> # remove a role from a user
aql> revoke role setB-user from fred
OK
aql> show user fred
+--------+-------------------------+
| user   | roles                   |
+--------+-------------------------+
| "fred" | "setA-user, user-admin" |
+--------+-------------------------+
1 row in set (0.000 secs)
OK
aql> # create a role with read-write privileges on namespaces "test" and "bar"
aql> create role new-role privileges read-write.test, read-write.bar
OK
aql> show role new-role
+------------+-----------------------------------+
| role       | privileges                        |
+------------+-----------------------------------+
| "new-role" | "read-write.bar, read-write.test" |
+------------+-----------------------------------+
1 row in set (0.000 secs)
OK
aql> # add a role to a user
aql> grant role new-role to fred
OK
aql> show user fred
+--------+-----------------------------------+
| user   | roles                             |
+--------+-----------------------------------+
| "fred" | "new-role, setA-user, user-admin" |
+--------+-----------------------------------+
1 row in set (0.000 secs)
OK
aql> # remove a privilege from a role (Affects any users who have the role)
aql> revoke privilege read-write.bar from new-role
OK
aql> show role new-role
+------------+-------------------+
| role       | privileges        |
+------------+-------------------+
| "new-role" | "read-write.test" |
+------------+-------------------+
1 row in set (0.000 secs)
OK
aql> # eliminate a role (Affects any users having the role)
aql> drop role new-role
OK
aql> show user fred
+--------+-------------------------+
| user   | roles                   |
+--------+-------------------------+
| "fred" | "setA-user, user-admin" |
+--------+-------------------------+
1 row in set (0.000 secs)
OK
```

## Local Passwords

Local users must have a valid password. For local passwords, these are valid password constructs:

- All alphabetic characters.
- All numeric characters.
- A mix of alphabetic and numeric characters.
- Contain a colon (:), hyphen (-), or underscore (_), but must begin with an alphanumeric character.
- Start with a hyphen, but the remaining characters must be numeric.
- Only contain the following comparison operators:
 - `<`
 - `>`
 - `=`
 - `>=`
 - `<=`
 - `<>`
- Only contain one of the following punctuation symbols:
 - `(`
 - `)`
 - `,`
 - `.`
 - `*`
 	
Invalid passwords:

- Begin with a hyphen and contain alphabetic characters.
- Begin with a colon and contain alphabetic characters.
- Contain comparison operators and other characters.
- Contain more than one punctuation symbol.
- Contain single quotes (') or double quotes (") or semicolon (;).

## Audit Trail

A separate audit trail exists on every server. Servers can be configured to generate audit log messages on the following actions:

- security violations
 - authentication failure
 - role violation
- successful authentications
- successful data transactions
- user administration operations
 - create, drop user
 - change user password
 - grant roles to user
 - revoke roles from user
 - create, drop role
 - grant privileges to role
 - revoke privileges from role
 - set whitelist for role (as of Aerospike 4.6)
 - query users and their roles
 - query roles and their privileges
- system administration operations
 - create, drop index
 - register, remove UDFs
 - set dynamic server configuration variables
 - enable specialized logging
 - get server configuration, server statistics, and other information

Servers can be configured to log audit messages to the following:

- syslog daemon default sink
- local syslog facility (local file)
- Aerospike log file

See this [syslog protocol definition](http://tools.ietf.org/html/rfc5424)

## Example Audit Trail Output

This is an example audit trail output. 

```sql
Feb 17 18:27:24 localhost asd: not authenticated | authenticated user: <none> | action: info request | detail: <none>
Feb 17 18:29:37 localhost asd: authentication failed (password) | authenticated user: <none> | action: authentication | detail: user=admin
Feb 17 18:30:45 localhost asd: permitted | authenticated user: admin | action: create user | detail: user=bruce;roles=read-write
Feb 17 18:46:10 localhost asd: permitted | authenticated user: bruce | action: write | detail: {test|test-set} [S|test-key]
Feb 17 18:53:23 localhost asd: permitted | authenticated user: admin | action: create role | detail: role=setA-writer;privs=rw{test|setA}
Feb 17 18:54:17 localhost asd: permitted | authenticated user: admin | action: create user | detail: user=sally;roles=setA-writer
Feb 17 19:15:08 localhost asd: permitted | authenticated user: sally | action: delete | detail: {test|setA} [D|16b7438da7d70e884fac0122ad79cdf213c1fc68]
Feb 17 19:15:54 localhost asd: role violation | authenticated user: sally | action: delete | detail: {test|setB} [D|ee50d7c1d0f427ed5c41ef8a18efd85412b973ff]
Feb 17 19:38:48 localhost asd: permitted | authenticated user: admin | action: create role | detail: role=setC-reader;privs=r{test|setC},r{bar|setC}
Feb 17 19:42:40 localhost asd: permitted | authenticated user: admin | action: delete privs | detail: role=setC-reader;privs=r{bar|setC}
Feb 17 19:44:40 localhost asd: permitted | authenticated user: admin | action: create role | detail: role=setB-user;privs=rwf{test|setB}
Feb 17 19:45:41 localhost asd: permitted | authenticated user: admin | action: query roles | detail: <none>
Feb 17 19:45:41 localhost asd: permitted | authenticated user: admin | action: read | detail: {test|test-set} [D|40737ffbe5ae0df603ed11b945b242d2577358a1]
```

In Aerospike Enterprise Edition revisions 3.7.0.1 or later, the audit trail output includes the IP address and port number of the client:

```sql
Oct 09 2018 04:22:36 GMT: INFO (security): (security.c:5482) permitted | client: 127.0.0.1:47692 | authenticated user: user1 | action: login | detail: user=user1
Oct 09 2018 04:22:36 GMT: INFO (security): (security.c:5482) permitted | client: 127.0.0.1:47694 | authenticated user: user1 | action: authentication | detail: user=user1
Oct 09 2018 04:22:38 GMT: INFO (security): (security.c:5482) permitted | client: 127.0.0.1:47694 | authenticated user: user1 | action: write | detail: {test|testset} [D|f59124986e96ad175b374c9487945bbcad537b74]
```

### Audit Trail Output Parsing

- The log message says in plain text what security action was attempted (if any), whether it succeeded or failed, and what user (if any) was involved.

- Namespaces and sets are in braces: `{namespace|set}` or just `{namespace|}`

- Custom role privileges are letter codes that correspond to Aerospike pre-defined roles, and are followed by `{namespace|set}` scoping (if applicable).

 The letter codes are:
 - `r` = read
 - `rw` = read-write
 - `rwf` = read-write-udf
 - `w` = write
 - `d` = data-admin (a global privilege, scoping not applicable)
 - `s` = sys-admin (a global privilege, scoping not applicable)
 - `u` = user-admin (a global privilege, scoping not applicable)

 For example:
 - `rw{}` indicates read-write privileges across all namespaces.
 - `r{ns1},rw{ns2}` indicates read privileges for the *ns1* namespace and read-write privileges for the *ns2* namespace.
 - `u,rwf{ns1|setA}` indicates user-admin privileges as well as read-write-udf privileges for *setA* in the *ns1* namespace.

- Key data is logged with successful data transactions (as well as with data transactions that fail due to role violations) if the key is stored. If the key is not stored, the digest is logged.

 Key data appears as:
 - `[S|mykey]` if the key is a string.
 - `[I|78654]` if the key is an integer.
 - `[B|AA C5 48]` if the key is a blob of bytes.

 If the key is not stored, the digest is logged as a string:
 - `[D|567895f996dd7dd3832222b57cd4a3031ecc6e24]`

## Security Configuration

To set up the Aerospike security feature:

1. Edit the *aerospike.conf* file to add a security section:

```bash
security {
    enable-security true

    # Write the audit trail to syslog (optional).
    syslog {
        local 0 # write to "local0" facility as well as to default syslog sink

        report-authentication true
        report-user-admin true
        report-sys-admin true
        report-violation true
        report-data-op test seta # report successful data transactions on set "seta" in namespace "test"
    }

    # Write audit trail to aerospike log file (optional).
    log {
        report-authentication true
        report-user-admin true
        report-sys-admin true
        report-violation true
        report-data-op test seta # report successful data transactions on set "seta" in namespace "test"
    }
}
```

2. (optional) Send the audit trail to a separate file using syslog, which can direct the audit trail to a separate file:

```
sudo vi /etc/rsyslog.conf
```

   Add the following:

```
 local0.*    /var/log/aerospike_security_audit.log
```

   Restart syslog (for example, for CentOS):

```
sudo /etc/init.d/rsyslog stop
sudo /etc/init.d/rsyslog start
```

3. (optional) Configure the audit trail to use the Aerospike log file by adding the following configuration to the *aerospike.conf* `syslog{}` section:

```bash
log {
    report-authentication true
    report-user-admin true
    report-sys-admin true
    report-violation true
    report-data-op test seta # report successful data transactions on set "seta" in namespace "test"
}
```
