{{!-- This page is used at /docs/operations/aws/--}}
{{!-- This page is used at /docs/operations/gcp/--}}

{{!-- Required variables: "persistence-name", "cloud-provider" --}}

This storage engine has persistence storage based on {{ persistence-name }} along with data stored in memory. This model provides for low latency quick access as all reads are from memory along with persistence provided by {{ persistence-name }}.
This has comparatively lesser RAM requirement than the In-memory only storage engine as it needs a replication factor of two only.
 
**Pros**:

- Speed of in-memory storage along with persistence storage provided by {{ persistence-name }}
- Comparatively less RAM and fewer number of nodes required.
- The suggested storage model in {{ cloud-provider }} for persistence requirement presently.

**Cons:**

- Requires additional {{ persistence-name }} storage which costs extra.
- Operational overhead of maintaining {{ persistence-name }} to node association.
- Need to setup own mechanism of snapshot/backup for further redundancy.

&nbsp; {{!-- Hack to not mess up the formatting later in the page --}}
