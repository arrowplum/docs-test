---
title: Access Control
description: Security and audit trail features in Aerospike Enterprise Edition.
assets: /docs/guide/assets
---

For database access management, administrators create users and assign passwords and roles. Each Aerospike client instance requires a user login. Login credentials are required for all server connections made by the client instance.

{{#info}}
[Cross-Datacenter Replication (XDR)](/docs/architecture/xdr.html) is compatible with the Aerospike Security feature as of Aerospike version 3.8. Previous versions of XDR are not compabile with the Aerospike Security feature.
{{/info}}

{{#warn}}
For Aerospike Server versions prior to 4.7 XDR does not support LDAP logins.
{{/warn}}

## Architecture

Aerospike is a high-performance and highly-available database. While Aerospike has traditionally focused on
low latency and high performance with minimal server resources, Aerospike had added features in the last few
years to improve capabilities in security.

Aerospike’s Authentication and Access Control system uses a standard three-layer architecture.
Users are created, which belong to groups, and groups have access rights such as admin, read table,
and write table permissions, among others.

Prior to Aerospike 4.1, the user authentication system is a pure user and password system,
with the user names and a hash of the password stored in the Aerospike cluster.
The client applies the same hash and sends a hashed value of the password across the network.

With Aerospike 4.1, the ability to use external authentication systems is supported. That release supports
LDAP authentication. An updated client will send not only the hashed version of the password (in case the user is internal),
but also the actual password, which will be sent to the external system. The external auth capabilities
can be disabled on the client login call, which will not send the password but only the hashed password.

TLS between client and server should be enabled when external authentication is used, since the plaintext password
must be sent for LDAP authentication. The password is **never** stored on the Aerospike server. If authentication
succeeds, a random number token is returned to the client, who may use it on subsequent TCP connections. The server
will time out these tokens according to a configured value, and require new TCP connections to be authenticated
externally.

In all cases, Aerospike currently maintains a local directory of both authorization and authentication.
This distributed database is maintained with a copy on every server for lowest read latency, and persists through local files.

After the initial phase of authentication, an authorization phase will create a query to the LDAP server which will
return a list of groups the user is a member of. This group list will be stored locally and distributed among all
cluster members, limiting load on the LDAP server.

This query will be done on a repeated basis by one cluster node, and thus allow changes in a user to be quickly
reflected to Aerospike, thus time-limiting a user’s access.

For users that are externally authenticated, there is no need to add that user manually to the Aerospike cluster in any way.

Support including both internal and external authentication is supported. An authentication attempt will include first internal validation, and then external validation. This is called “hybrid authentication”.

## LDAP Direct Bind vs Query User DN authentication

The Aerospike server takes a username and maps it to an LDAP DN. The DN is then used for
the authentication (bind) request as well as querying for groups in some cases.

If the username is not in the DN (often the case for Active Directory) or if user DNs are in
different OUs, a “user-query-pattern“ must be configured. This query must return exactly one user record
from which the DN is extracted. For example in Active Directory the configuration would be something like:

```
user-query-pattern (&(objectCategory=Person)(sAMAccountName=${un}))
```

The username gets substituted in for the “${un}” so in the above case if the username was “janet_doe”
the actual query would be “(&(objectCategory=Person)(sAMAccountName=janet_doe))”  which, when run,
would return the entry with a DN of “cn=Janet Doe,ou=People,dc=janetsoft,dc=com”

In the special case where the system’s user’s DNs always follow the same pattern with
the username embedded (eg “uid=johnny_doe,ou=People,dc=johnnycorp,dc=com”), it will save
one extra query to configure the “user-dn-pattern” directly instead of the user-query-pattern.
For example in johnnycorp’s case the configuration parameter would be:
```
    user-dn-pattern uid=${un},ou=People,dc=johnnycorp,dc=com
```

In this case no extra user lookup query will be made and the substituted DN will be used.


## External Authentication (LDAP) group mapping

In order to provide a cleaner management solution, Aerospike will use the external server ( LDAP )
to provide the mapping from an individual username to a list of groups. This is done through a set of LDAP queries.
The Aerospike server configuration is described in the section LDAP Users and Groups below.

The results of this request will be stored in Aerospike’s shared SMD ( System Metadata store ).
This allows the query responses to be shared among all Aerospike servers, reducing the query load.
This request will be done on a repeated basis ( configurable time ). If the response shows that the user has
been deleted or the user has been modified, all servers will be quickly notified of the change, and all
authorization will use the new authorization – including existing in-flight connections.

Aerospike’s SMD store also includes the time that the data was last updated. Aerospike will be configured
to not use data past a certain age, providing some resiliency in case of LDAP server failure, but also providing
protection in case of an LDAP DOS attack.

Through this mechanism, changes in user authorization and authentication will be quickly learned and used throughout
the cluster, on a time-bounded basis.

The mapping of groups to individual access rights will be done through existing Aerospike configuration mechanisms,
and use the existing Aerospike logging and levels. An administrator will need to create a set of groups, and
determine the privilege levels of those groups, and set those groups in Aerospike using the already existing and
provided tools.

#### LDAP Users and Groups

Organizing users and giving permissions in LDAP is flexible. The supported groupings to use for Aerospike roles
fit into two categories:

**Groupings of users** These groupings can be posix groups, OUs like “groupOfUniqueNames”, or any
other entry that can be queried using either username or DN of the user.
The role-query-pattern(s) are used to find these groups :

**role-query-pattern** contains an LDAP query to search for an entry. Multiple may be specified for
cases where groups are kept in different locations or need different queries. The queries will be run in
order and all entries found will be collected.

Example configuration might be:

```
role-query-pattern (&(objectClass=posixGroup)(memberUid=${un}))
role-query-pattern (&(ou=db_groups)(uniqueMember=${dn}))
role-query-pattern (&(ou=admins)(uniqueMember=uid=${un},dc=blastcorp,dc=com))
```

Note the substitutions for username, ${un}, and distinguished name, ${dn}. These will be replaced
by the actual username and the actual user’s full distinguished name when the queries get run.

**User’s position in LDAP tree**  In some setups the user’s position within a directory tree
is what determines their privilege level. For example a user such as
``uid=jim,ou=Users,ou=Admins,dc=blastcorp,dc=com`` might have groups of privileges for “Users” as well as for
“Admins”. If this is the case, setting role-query-search-ou to true will treat the OUs in the user’s DN as potential roles.

In all cases the CN of the entry must match an Aerospike role name to be detected as such; any
others will be ignored.


## Configuration

Refer to the [Access Control Configuration Guide](/docs/operations/configure/security/access-control/index.html) for specific
instructions for configuring access control on your cluster.

Refer to the [LDAP Configuration Guide](/docs/operations/configure/security/ldap/index.html) for specific
instructions for configuring LDAP on your cluster.

## Users

As of Aerospike 4.1, users can reside either in the internal Aerospike user database - which is distributed and shared among
cluster members - or externally authenticated users. Users operate in a hybrid mode, where the internal database is checked
first, and if no user is found, the external authentication system is invoked, and the username and password will be
handed to the external system for authentication.

Aerospike internal users are created and managed using the user *admin*. By default, the
initial *admin* password is *admin* and is the user-admin role.

{{#warn}}
Change the *admin* password immediately after enabling security.
{{/warn}}

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
| user-admin  <br> <br> <br> <br> <br> <br> <br> <br> <br> <br> <br>|- create and drop users <br>- change any user password  <br>- grant roles to users  <br>- revoke user roles <br>- create and drop user roles <br>- grant user role privileges <br>- revoke role privileges <br>- set whitelists for roles (as of Aerospike 4.6) <br>- query all users and their roles  <br>- query all roles and their privileges  <br>- get server configuration and statistics |

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


## Passwords

Users must have a valid password. For internal user passwords, these are valid password constructs:

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

Refer to the [syslog protocol definition](http://tools.ietf.org/html/rfc5424) document.

