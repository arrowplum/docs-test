---
title: Using Policies
description: Use policies to define behavior for the database operations with the Aerospike C client.
---

The Aerospike C client uses policies to define behavior for database operations. Policies are values that dictate the behavior of an operation. Each operation relies on a set of policy values, collectively called an _operation_ policy. Each operation accepts a policy as the third argument. For example, `aerospike_key_get()` requires an `as_policy_read` as the third argument.

```cpp
as_status aerospike_key_get(
    aerospike *as, as_error *err, const as_policy_read *policy, 
    const as_key *key, 
    as_record **rec
    );
```

### Policy Values

A policy is a value used by an operation. Policies can also have an undefined value. For example, the _timeout_ policy is considered undefined when its value is 0. Also, the _key_ policy is undefined when its value is `AS_POLICY_KEY_UNDEFINED`. When a value is undefined, it triggers a fallback mechanism to find a default value for the policy. 

### Policy Fallback Mechanism

When a policy value is undefined for an operation or the operation policy is NULL, then the undefined policy values default to those in the operation policy for that operation. The default operation policy is defined as part of the Aerospike C client instance, which dictates default behavior for that client instance. If the default operation policy contains an undefined value, then the value in the default policy is used. Default policy values are defined in the Aerospike C client.

For example, `as_policy_read` is an operation policy applicable to read operations. This policy has two policy values: _timeout_ and _key_. The _timeout_ policy dictates operation wait time before a timeout. The _key_ policy (`as_policy_key`) dictates how to use the key in the operation. A _timeout_ of 0 (zero) is considered an undefined value. The _key_ policy also has a undefined value: `AS_POLICY_KEY_UNDEFINED`. 

The `aerospike_key_get()` operation accepts an `as_policy_read` policy. The following example sets the operation policy to NULL to return all  values for the policy to the values in the default operation policy.

```cpp
aerospike_key_get(&as, &err, NULL, &key, &rec);
```

In this example, the default operation policy for read (`as_policy_read`) is defined 
as:

```cpp
aerospike.config.policies.read.timeout = 0;
aerospike.config.policies.read.key = AS_POLICY_KEY_DIGEST;
```

The _key_ policy is now defined as `AS_POLICY_KEY_DIGEST`, but _timeout_ is still undefined to cause it to fallback to the default policy value for _timeout_, which is defined as:

```cpp
aerospike.config.policies.timeout = 1000;
```

This means that the actual policy used for the `aerospike_key_get()` operation is:

```cpp
policy.timeout = 1000;
policy.key = AS_POLICY_DIGEST;
```

### Defining Default Policies

Default policies are defined during client configuration. This allows you to set the default policies prior to initializing the Aerospike C client. 

{{#note}}
You must initialize default policies to their default values before modifying the configuration. Once initialized, you can modify the policies.
{{/note}}

```cpp
as_config config;
as_config_init(&config)
```

To change the default _timeout_ value for all operations to 2 seconds:

```cpp
config.policies.timeout = 2000;
```

To set write operations to a 1-second timeout, but keep all other operations to 2-second timeouts:

```cpp
config.policies.write.timeout = 1000;
```

In this example, `aerospike_key_get()` defaults to a 2-second timeout and `aerospike_key_put()` defaults to a 1-second timeout.

### Defining a Policy for an Operation

To provide an operation policy on a per-operation basis, initialize the policy and set the values to override. Note that each operation policy has an initialization function.

```cpp
as_policy_read policy;
as_policy_read_init(&policy);
```

Then, set the values of the policy:

```cpp
policy.timeout = 5000;
```

Now, pass the policy to the operation:

```cpp
aerospike_key_get(&as, &err, &policy, &key, &rec);
```

This results in the following policy for the operation:

```cpp
policy.timeout = 5000;
policy.key = AS_POLICY_DIGEST;
```

This is because the _key_ policy value was not defined for the policy provided to the operation. It defaults to the default read policy value for _key_ (as previously defined):

```cpp
aerospike.config.policies.read.key = AS_POLICY_KEY_DIGEST;
```

