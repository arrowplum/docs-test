---
title: Upgrading from Community to Enterprise version
description: Learn the proper procedures to upgrade the Community Version of Aerospike to Enterprise Release. This procedure will ensure there is no downtime required for the upgrade.
styles:
  - /assets/styles/ui/steps.css
---

Aerospike Enterprise Edition includes additional features and provides access to tested and certified builds, hot patches and 24x7 Enterprise Support.
To see all other benefits of using the Enterprise Edition, go to [Products Pricing](https://www.aerospike.com/products-services/)

{{#info}}
Note: Please contact **Aerospike support** if you need clarification on any
of the following steps.
{{/info}}

You can use the following steps to upgrade a cluster of community-version nodes to the enterprise version. For the purposes of upgrading, you can add an enterprise node to a cluster of community nodes.

{{#steps}}

Perform the following steps on **each** node, **one node** at a time.

<br></br>
{{#steps-step 1 "Check Migrations" markdown=true}}

Verify that the cluster is not migrating data before you stop one of the nodes. Run the following command to check migration status:

   ```bash
   # Migrations are complete when all `migrate_progress_recv` and
   # `migrate_progress_send` values return 0.
   asadm -e "show statistics like migrate_progress"
   ``` 
{{/steps-step}}
{{#steps-step 2 "Stop the Aerospike service" markdown=true}}
1. Run the following command to display the current status of the Aerospike Service:
   
   ```bash
   sudo service aerospike status
   ```

2. Run the following command to stop the Aerospike service on the node:

    ```bash
    sudo service aerospike stop
    ```

{{/steps-step}}

{{#steps-step 3 "Backup the Aerospike Configuration File" markdown=true}}

Create a backup of the current configuration file:
```bash
sudo cp /etc/aerospike/aerospike.conf /etc/aerospike/aerospike.conf.bac
```
{{/steps-step}}

{{#steps-step 4 "Uninstall the Community Aerospike Server Package" markdown=true}}

- For **Redhat/Centos** distributions

    ```bash
    sudo rpm -e aerospike-community-server-<VERSION>.rpm
    ```
    ```bash
    #If you have tools package installed separately
    sudo rpm -e aerospike-community-tools-<VERSION>.rpm
    ```

- For **Debian/Ubuntu** distributions:<br>

    ```bash
    sudo dpkg -r aerospike-community-server-<VERSION>.deb
    ```
    ```bash
    #If you have tools package installed separately
    sudo dpkg -r aerospike-community-tools-<VERSION>.deb
    ```

{{/steps-step}}

{{#steps-step 5 "Install Aerospike Enterprise packages" markdown=true}}

**Download Aerospike** and **Install Aerospike**. For more information on downloading and installing, see the [Installation Documentation](/docs/operations/install).

{{#info}}
You must login to download the Enterprise installer files. For queries regarding access, please contact your **Sales Rep**.
{{/info}}

{{/steps-step}}

{{#steps-step 6 "Restore the backup configuration" markdown=true}}

Restore the backup configuration file to **`/etc/aerospike/aerospike.conf`**:
   ```bash
   sudo cp /etc/aerospike/aerospike.conf.bac /etc/aerospike/aerospike.conf
   ```
{{/steps-step}}

{{#steps-step 7 "Start Aerospike server" markdown=true}}

The following steps ensure that you have installed the Enterprise Aerospike server and finished migrations on each
node before proceeding to next node.

1. Start the Aerospike service. Run the following command:

    ```bash
    sudo service aerospike start
    ```

    For more information on starting the service, see the [Installation     Documentation](/docs/operations/install).
   
   {{#info}}
   Note: If you see warnings in the Aerospike log about deprecated parameters, confirm their usage in Aerospike 3 in the [Configuration Reference](/docs/reference/configuration).
   {{/info}}

2. Wait for Aerospike's service port to open (port 3000) before continuing.
   When the service port opens, the following command will return "OK":
   ```bash
   asinfo -v STATUS
   ```

3. If your server contains any namespaces that are **in-memory without
   persistence**, you must wait for migrations to complete before continuing
   to the next node to prevent data loss. 
   
   Run the following command to verify migrations are complete:
   ```bash
   # Migrations are complete when all `migrate_progress_recv` and
   # `migrate_progress_send` values return 0.
   asadm -e "show statistics like migrate_progress"
   ```

4. If all namespaces are **persisted**, you may continue to upgrade the next node and not lose any data.
   {{#warn}}
   If writes are still active on the cluster, and you choose to not wait for migrations to complete between node upgrades, 
   some record updates may be overwritten during migration conflict resolution. Contact Aerospike Support for further details if necessary.
   {{/warn}}

{{/steps-step}}

{{#steps-step 8 "Verify Aerospike server" markdown=true}}

After you complete the steps on each node, use these [Steps](/docs/operations/verify) to confirm that Aerospike was successfully installed and is running.

{{/steps-step}}

{{/steps}}
