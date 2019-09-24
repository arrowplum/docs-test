---
title: Configuring LDAP
description: Configure LDAP
---

## LDAP Configuration

### Client version requirements

{{#warn}}
For Aerospike Server versions prior to 4.7 XDR does not support LDAP logins.
{{/warn}}

To use the External Authentication system described below, please be aware of the minimum client version levels required:
- Java 4.1.4
- C / C++ 4.3.6
- C# .NET 3.6.1
- Python 3.1.0

The API for authentication has not changed. If the client is not up to date, only Aerospike local users will be
available for login.

### Configuration Options

Refer to the [Access Control guide](/docs/guide/security/access-control.html) for basic feature concepts.

LDAP is an Enterprise Edition feature.
* For Aerospike Server version 4.1 & 4.2, a [`feature-key-file`](/docs/reference/configuration#feature-key-file) with *ldap* set to *true* is required.
* For Aerospike Server version 4.3 and newer, a [`feature-key-file`](/docs/reference/configuration#feature-key-file) with *asdb-ldap* set to *true* is required.

The default location of the feature-key-file is `/etc/aerospike/features.conf`.  
Contact Aerospike sales/support if you do not have a feature-key.

LDAP support is activated by the [enable-ldap](/docs/reference/configuration#enable-ldap) (can be set to true or false) option in the security section of the aerospike configuration file.  The default setting is false.

The other LDAP option in the security section is `[ldap-login-threads](/docs/reference/configuration#ldap-login-threads)` (integer) which specifies the number of threads to use for LDAP logins.  The allowable range is 1 to 64.  Default: 8

All other LDAP options belong to the ldap subsection of the security section of the aerospike configuration file.

Dynamic options can be changed at run time via an info command, e.g. `asinfo -v 'set-config:context=security;ldap.polling-period=3600'`

Note: Option values may not contain any spaces; however, if a space is needed in role-query-pattern, or user-query-pattern, \20 may be used instead of a space.

{{#note}}
Important: The default setting for the authentication from the client side is 'INTERNAL' (this is used when the hashed password is stored in the server and can be used without TLS). In order to use LDAP, **the authentication mode needs to be set to 'EXTERNAL'** (this is used when specific external authentication is configured on the server). If TLS is defined, a clear password is sent on node login via TLS and an exception is thrown if TLS is not defined). The third option for authentication is 'EXTERNAL_INSECURE' which can be used for testing purposes (this mode would send the password in clear text with or without TLS).
{{/note}}

### LDAP Options

**[`disable-tls`](/docs/reference/configuration/#disable-tls)** `<true|false>` whether to disable the use of TLS for LDAP server connections.  Default: false

**[`polling-period`](/docs/reference/configuration/#polling-period)** ``<integer>`` how frequently (in seconds) to query the LDAP server for user group membership information.
Allowable range is 0 to 86400 (24 hours).  Note that a value of 0 means do not poll.  Default: 300 (5 minutes).  Dynamic.

**[`query-base-dn`](/docs/reference/configuration/#query-base-dn)** ``<string>`` distinguished name of the LDAP directory entry at which to begin the search when querying for a
user's group membership information.  Required option.

**[`query-user-dn`](/docs/reference/configuration/#query-user-dn)** ``<string>`` distinguished name of the user designated for user group membership queries.

**[`query-user-password-file`](/docs/reference/configuration/#query-user-password-file)** ``<string>`` path to file containing clear text password for the user designated for user
group membership queries.

**[`role-query-base-dn`](/docs/reference/configuration/#role-query-base-dn)** ``<string>`` if specified uses this value as the base dn when performing the role queries.
Default: query-base-dn value is used

**[`role-query-pattern`](/docs/reference/configuration/#role-query-pattern)** ``<string>`` – format for the search filter to use when querying for a user's group membership
information. The substitutions for username, ${un}, and distinguished name, ${dn} will be replaced by the actual
username and the actual user’s full distinguished name when constructing the search filter.  If needed, multiple role-query-pattern
strings can be specified separately and each will be tried in order when querying for a user's information.  Required option.

**[`role-query-search-ou`](/docs/reference/configuration/#role-query-search-ou)** `<true|false>` whether to look for a user's group membership information in the organizational unit entries of the user's LDAP distinguished name.  Default: false

**[`user-dn-pattern`](/docs/reference/configuration/#user-dn-pattern)** `<string>` format for the distinguished name of the LDAP directory entry to use when binding to the LDAP
server for user authentication. ${un} should be placed in this string to specify where the user ID is inserted when
constructing the distinguished name.  Either this option or user-query-pattern is required.

**[`user-query-pattern`](/docs/reference/configuration/#user-query-pattern)** `<string>` format for the search filter to use when querying for a user's distinguished name.
${un} should be placed in this string to specify where the user ID is inserted when constructing the search filter.
Either this option or user-dn-pattern is required.

**[`server`](/docs/reference/configuration/#server)** `<string>` name of the LDAP server to use.  Multiple servers can be specified via a comma-delimited string without whitespace.  Required option. LDAPS support only available for server version 4.5.3.5 and above (also available on 4.5.2.5, 4.5.1.10, 4.5.0.14).

**[`session-ttl`](/docs/reference/configuration/#session-ttl)** `<integer>` lifetime (in seconds) of an access token.  A TCP connection attempt with an expired token will fail, and the client must log in again to get a fresh token.  Allowable range is 60 (1 minute) to 864000 (10 days).  Default: 18000 (5 hours).  Dynamic.

**[`tls-ca-file`](/docs/reference/configuration/#tls-ca-file)** `<string>` path to the CA certificate file used for validating TLS connections to the LDAP server.  Includes filename, e.g. /path/to/CA/cert/filename.  Required option unless disable-tls is true.

**[`token-hash-method`](/docs/reference/configuration/#token-hash-method)** `<string>` hash algorithm to use when generating the HMAC for access tokens.  Currently supported algorithms are sha-256 and sha-512.  Default: sha-256

### Example configurations

Below are examples of LDAP configuration, as used in development when testing.

Please note: These are just examples and must be modified to fit your particular setup.

Example using JumpCloud's LDAP directory-as-a-service

```
security {
    enable-security true
    enable-ldap true

    ...

    ldap {
    # base dn will be different for every jumpcloud instance
        query-base-dn o=5a7b4b08f4f3414e69eb5f8c,dc=jumpcloud,dc=com
    # ldap server(s) to use
        server ldap://ldap.jumpcloud.com:389
    # this is the default, but we want to use tls
        disable-tls false
    # location of the cert file needed for TLS communication with the jumpcloud server
        tls-ca-file /etc/ssl/certs/jumpcloud.chain.pem
    # this is the dn of the aerospike "trusted user" which is used to query for roles
        query-user-dn uid=as_user,ou=Users,o=5a7b4b08f4f3414e69eb5f8c,dc=jumpcloud,dc=com
    # path to cleartext password of the same user
        query-user-password-file /some/path/to/the_query_user_password.txt
    # set this to true to treat the OUs of the users dn as possible roles
        role-query-search-ou true
    # this query will be used to search for other entries to use for roles.
    # in this case the ${dn} will be substituted with the user's full dn
        role-query-pattern (&(objectClass=groupOfNames)(member=${dn}))
    # how often to poll for new roles. We are doing it every 10 seconds for development testing
        polling-period 10
    # pattern to look up user. ${un} will be replaced with the username
        user-query-pattern (uid=${un})
    }

    ...

}
```

Example using Active Directory

Query user DN, no TLS, trusted role poling user, and multiple group types

```
security {
    enable-security true
    enable-ldap true
    ldap {
        query-base-dn dc=aerospike,dc=com
        server ldap://blast.aerospike.com:389
        # This server does not support TLS
        disable-tls true
        query-user-dn CN=Trusted\20User,CN=Users,DC=aerospike,DC=com
        query-user-password-file  /path/to/password_file.txt
        role-query-search-ou true
        role-query-pattern (&(objectClass=posixGroup)(memberUid=${un}))
        role-query-pattern (&(objectClass=group)(member=${dn}))
        role-query-pattern (&(ou=*)(uniqueMember=uid=${un},dc=aerospike,dc=com))
        polling-period 20
        user-query-pattern (sAMAccountName=${un})
    }
    log {
        report-violation true
    }
}
```

Example OpenLDAP Configurations

Using direct bind pattern, TLS, anonymous role binding, and posix groups

```
security {
    enable-security true
    enable-ldap true
    ldap {
        query-base-dn dc=aerospike,dc=com
        server ldap://localhost:389
        tls-ca-file /etc/ssl/certs/ca_server.pem
        user-dn-pattern uid=${un},ou=People,dc=aerospike,dc=com
        role-query-search-ou true
        role-query-pattern (&(objectClass=posixGroup)(memberUid=${un}))
        polling-period 10
    }
    log {
        report-violation true
    }
}
```

Query user DN, TLS, trusted role polling user, and multiple group types

```
security {
    enable-security true
    enable-ldap true
    ldap {
        query-base-dn dc=aerospike,dc=com
        server ldap://ldaptest.aerospike.com:389,ldap://localhost:389
        tls-ca-file /etc/ssl/certs/ca_server.pem
        query-user-dn uid=aero,dc=aerospike,dc=com
        query-user-password-file /path/to/password_file.txt
        role-query-pattern (objectClass=group)
        role-query-pattern (&(ou=*)(uniqueMember=uid=${un},dc=aerospike,dc=com))
        role-query-pattern (&(objectClass=posixGroup)(memberUid=${un}))
        polling-period 10
        user-query-pattern (&(uid=${un})(|(objectClass=inetOrgPerson)(objectClass=posixAccount)))
    }
    log {
        report-violation true
    }
}
```

### LDAP Debugging Tips

#### Using `ldapsearch`

When configuring LDAP in Aerospike, especially in writing your queries (filters in LDAP) it is very
helpful to have the full ldap records for both a typical ldap user and the group(s) to use for Aerospike roles.

The easiest way to get this information is with the `ldapsearch` tool. An example `ldapsearch` query for a user
record could be:
```
ldapsearch -w password -D CN=trusted_user,OU=ServiceAccount,DC=example,DC=com -b dc=example,dc=com -h ldap://ldap.example.com:389  '(uid=my_user)'
```

For a group it might be:
```
ldapsearch -w password -D CN=trusted_user,OU=ServiceAccout,DC=example,DC=com -b DC=example,DC=com -h ldap://ldap.example.com:389  '(CN=some_group)'
```

If you have TLS enabled you must add your certs on the client machine and either an ldap.conf file must be created, or
environment variables must be set. See https://www.openldap.org/doc/admin24/tls.html for more details

`ldapsearch` is also a great tool to test your query patterns. The pattern can be plugged directly to the earlier queries
(replacing the `(CN=some_group)` in the above example) but with a value substituted in for the `${un}` placeholder.

For example:

If you have a config like `role-query-pattern (&(objectClass=posixGroup)(memberUid=${un}))` you can see if the query is
valid for a perspective user with a uid of `joe_user` by issuing a command like:

```
ldapsearch -w password -D CN=trusted_user,OU=ServiceAccout,DC=example,DC=com -b DC=example,DC=com -h ldap://ldap.example.com:389  '(&(objectClass=posixGroup)(memberUid=joe_user))'
```

#### Extra logging

While setting up ldap it might be useful to turn on some extra logging to see what role queries are being made for login or polling and other verbose information. To do that you may either specify
```
    console {
        context security detail
    }
```
or
```
    file <filename> {
        context security detail
    }
```

in the logging section of your config. If you would like to activate detailed security logging dynamically in a running system you may execute the command:
```
    asinfo -v 'log-set:security=detail'
```


