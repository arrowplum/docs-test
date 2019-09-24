{{!-- This page is used at /docs/deploy_guides/aws/plan/ --}}
{{!-- This page is used at /docs/deploy_guides/gcp/plan/ --}}

{{!-- Required variables: "persistence-name", "cloud-provider" --}}

The two most common use-cases for Aerospike are:
* An in-memory cache with no persistence
* A fast persistent data store

We will look at how to implement these cases on the {{ cloud-provider }} platform. For information on more choices to organize your storage, please see the [storage engines](/docs/operations/configure/namespace/storage) documentation.
