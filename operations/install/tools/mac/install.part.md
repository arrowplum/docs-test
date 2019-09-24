<a name="install"></a>
### Install Aerospike Tools
If you have already installed the Tools, you can skip directly to **[Validate Aerospike Tools Installation](#validate)**.

{{#steps}}
{{#steps-step 4 "Install the package on Mac" markdown=true}}

Please check the requirements [here](/docs/operations/install/tools/requirements) and install if any package is missing.

You can install the downloaded package by double clicking the package file and following the instructions. This should install the Aerospike tools package on your Mac.

{{#note}} Tools installation installs PEX binary for `asadm` tool which might be slow in start-up phase due to PEX packaging overhead. For faster startup times, please install the non-pex version of `asadm` from [source](https://github.com/aerospike/aerospike-admin). {{/note}}

{{/steps-step}}
{{/steps}}


