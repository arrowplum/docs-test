{{!-- This page is used at /docs/operations/aws/--}}
{{!-- This page is used at /docs/operations/gcp/--}}

{{!-- Required variables: "persistence-name", "cloud-provider" --}}

In this storage engine, only the index resides in memory and data is stored in {{ persistence-name }}.

**Pros:**
 
- Comparatively less RAM and fewer number of nodes required.
- Can make use of instances with lesser RAM.
- Provides persistence model.

**Cons:**

- The maximum possible TPS is potentially bottlenecked by maximum available {{ persistence-name }} IOPS.
- Further degradation of service due to {{ persistence-name }} performance uncertainties is possible.

&nbsp; {{!-- Hack to not mess up the formatting later in the page --}}
