<a name="forget"></a>
### Forget Old Tools Packages
If you have already installed a previous version of Tools, you will need to "forget" any previously installed versions of Tools in order to avoid potential conflicts.

{{#steps}}
{{#steps-step 2 "Forget Old Tools Packages" markdown=true}}

First grep to see what version of Tools is previously installed by using the following command:
'''bash
$ pkgutil --pkgs | grep 'aerospike.tools'
'''

---
The above command should output the tools package installed
sample:
```bash
$ pkgutil --pkgs | grep 'aerospike.tools'
com.aerospike.tools.3.13.0.1
``` 

On a Mac machine, use the following command to forget any previous installations:
``` bash
$ sudo pkgutil --forget com.aerospike.tools.x.xx.x
``` 

Note that x.xx.x represents your version number.
---
{{/steps-step}}
{{/steps}}


