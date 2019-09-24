---
title: Deploy an Aerospike Cluster using Helm Charts and local volume static provisioner
description: |
  Using Helm Charts to deploy an Aerospike Cluster on Kubernetes with namespace data storage on local SSDs as block devices with automated local volume provisioning.
scripts:
  - /assets/scripts/utils/target_blank.js

---

### Background

In this example, we will be using **Helm Charts** to deploy an Aerospike Cluster on Kubernetes with namespace data storage on **local SSDs** as **block devices** with **automated local volume provisioning**.

### Steps

1. Create a discovery directory on each host from which the provisioner will discover local block volumes.
    ```sh
    $ mkdir /mnt/disks
    ```

2. Link the devices into the discovery directory.

    - We will be using the below two devices `/dev/sdb` and `/dev/sdc` for our namespace storage from each host.
    ```sh
    $ lsblk 
    NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    sdb       8:16   0  375G  0 disk 
    sdc       8:32   0  375G  0 disk 
    ```
    - For safety purpose, we will use the device by it's unique ID rather than `/dev/sdb` or `/dev/sdc`
    ```sh
    $ ln -s /dev/disk/by-id/local-ssd-0 /mnt/disks
    $ ln -s /dev/disk/by-id/local-ssd-1 /mnt/disks
    ```

    Perform the steps for each node in the Kubernetes cluster. In this example, there are two nodes, each with two local SSDs attached.

3. To automate the local volume provisioning, we will create and run a local volume provisioner based on [kubernetes-sigs/sig-storage-local-static-provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner). The provisioner will run as a DaemonSet which will manage the local SSDs on each node based on the discovery directory (created in Step 1), create/delete the `PersistentVolumes` and clean up the storage when it is released.
    - Create a file `aerospike-local-volume-provisioner.yaml` with below specifications:

      - ServiceAccount for the provisioner
      - ClusterRole and ClusterRoleBindings (to create and delete PV objects)
      - A ConfigMap object for the provisioner
      - DaemonSet for running the provisioner
      
      ```sh
      ### ========================================
      ### aerospike-local-volume-provisioner.yaml
      ### ========================================

      # Provisioner ConfigMap
      #
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: local-provisioner-config
        namespace: default
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      data:
        useNodeNameOnly: "true"
        storageClassMap: |
          aerospike-ssds:
            hostDir: /mnt/disks
            mountDir: /mnt/disks
            blockCleanerCommand:
              - "/scripts/shred.sh"
              - "2"
            volumeMode: Block
      ---#

      # Daemonset
      # Local volume Provisioner
      #
      apiVersion: apps/v1
      kind: DaemonSet
      metadata:
        name: local-volume-provisioner
        namespace: default
        labels:
          app: local-volume-provisioner
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      spec:
        selector:
          matchLabels:
            app: local-volume-provisioner
        template:
          metadata:
            labels:
              app: local-volume-provisioner
          spec:
            serviceAccountName: local-storage-admin
            containers:
              - image: "quay.io/external_storage/local-volume-provisioner:v2.3.3"
                name: provisioner
                securityContext:
                  privileged: true
                env:
                - name: MY_NODE_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: spec.nodeName
                - name: MY_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
                - name: JOB_CONTAINER_IMAGE
                  value: "quay.io/external_storage/local-volume-provisioner:v2.3.3"
                volumeMounts:
                  - mountPath: /etc/provisioner/config
                    name: provisioner-config
                    readOnly: true
                  - mountPath: /mnt/disks
                    name: aerospike-ssds
                    mountPropagation: "HostToContainer"
            volumes:
              - name: provisioner-config
                configMap:
                  name: local-provisioner-config
              - name: aerospike-ssds
                hostPath:
                  path: /mnt/disks
      ---#

      # Storage Class 'aerospike-ssds'
      #
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: aerospike-ssds
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      provisioner: kubernetes.io/no-provisioner
      volumeBindingMode: WaitForFirstConsumer
      reclaimPolicy: Delete

      ---#

      # Provisioner Service Account
      #
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: local-storage-admin
        namespace: default
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3

      ---#

      # Provisioner Cluster Role Binding
      #
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: local-storage-provisioner-pv-binding
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      subjects:
      - kind: ServiceAccount
        name: local-storage-admin
        namespace: default
      roleRef:
        kind: ClusterRole
        name: system:persistent-volume-provisioner
        apiGroup: rbac.authorization.k8s.io
      ---#

      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRole
      metadata:
        name: local-storage-provisioner-node-clusterrole
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      rules:
      - apiGroups: [""]
        resources: ["nodes"]
        verbs: ["get"]
      ---#

      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRoleBinding
      metadata:
        name: local-storage-provisioner-node-binding
        labels:
          heritage: "Tiller"
          release: "RELEASE-NAME"
          chart: provisioner-2.3.3
      subjects:
      - kind: ServiceAccount
        name: local-storage-admin
        namespace: default
      roleRef:
        kind: ClusterRole
        name: local-storage-provisioner-node-clusterrole
        apiGroup: rbac.authorization.k8s.io
      ---#
      ```
    - Deploy the provisioner,
        ```sh
        $ kubectl create -f aerospike-local-volume-provisioner.yaml

        configmap/local-provisioner-config created
        daemonset.apps/local-volume-provisioner created
        storageclass.storage.k8s.io/aerospike-ssds created
        serviceaccount/local-storage-admin created
        clusterrolebinding.rbac.authorization.k8s.io/local-storage-provisioner-pv-binding created
        clusterrole.rbac.authorization.k8s.io/local-storage-provisioner-node-clusterrole created
        clusterrolebinding.rbac.authorization.k8s.io/local-storage-provisioner-node-binding created
        ```

4. Run `kubectl get pv` to verify the discovered and created PV objects. Note that each PV (PersistentVolume) object's status is `Available`. Once bound to a PVC(PersistentVolumeClaim) and in active use by a pod, the status will change to `Bound`.
    ```sh
    $ kubectl get pv
    NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS     REASON   AGE
    local-pv-342b45ed   375Gi      RWO            Delete           Available           aerospike-ssds            12m
    local-pv-3587dbec   375Gi      RWO            Delete           Available           aerospike-ssds            12m
    local-pv-df716a06   375Gi      RWO            Delete           Available           aerospike-ssds            12m
    local-pv-eaf4a027   375Gi      RWO            Delete           Available           aerospike-ssds            12m
    ```

5. Now we can go ahead and configure to use these storage volumes in our deployment. In this example, we will simply edit the [values.yaml](https://github.com/aerospike/aerospike-kubernetes-enterprise/blob/master/helm/values.yaml) file to add the `persistenceStorage` configuration section as shown below. Note that the `storageClass` here must point to `aerospike-ssds` (as defined in Step 3). We have also set `volumeMode` to `Block` to use the block devices directly for Aerospike namespace data storage.

    ```sh
    ....
    ### Deployment specific configurations

    persistenceStorage:
      - devicePath: /dev/xvdk
        enabled: true
        name: data-dev
        storageClass: aerospike-ssds
        accessMode: ReadWriteOnce
        size: 375Gi
        volumeMode: Block
    ...
    ```

6. Configure [aerospike.conf](https://github.com/aerospike/aerospike-kubernetes-enterprise/blob/master/helm/files/aerospike.template.conf) file or template to use the the raw block device `/dev/xvdk` in the namespace `storage-engine` configuration. You can also configure and pass in your own aerospike.conf file or template.

    ```sh
    ...
    namespace ${NAMESPACE} {
      replication-factor ${REPL_FACTOR}
      memory-size ${MEM_GB}G
      default-ttl ${DEFAULT_TTL}

      storage-engine device {
        device /dev/xvdk
        write-block-size 128K
      }
    }
    ```

7. Install the chart by passing in the **new** `aerospike.template.conf` and **new** `values.yaml` file.
    ```sh
    helm install \
    --set dBReplicas=4 \
    --name as-release aerospike/aerospike-enterprise \
    --set-file featureKeyFilePath=/secrets/aerospike/features.conf \
    --values helm/values.yaml \
    --set-file confFilePath=helm/files/aerospike.template.conf
    ```

8. Once the deployment is successful, we can verify the status of these volumes, and it is now `Bound` and is in active use by the Aerospike pods.

    ```sh
    $ helm status as-release
    LAST DEPLOYED: Mon Sep 16 22:45:54 2019
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/ConfigMap
    NAME             DATA  AGE
    as-release-conf  4     25m

    ==> v1/Pod(related)
    NAME                               READY  STATUS   RESTARTS  AGE
    as-release-aerospike-enterprise-0  1/1    Running  0         25m
    as-release-aerospike-enterprise-1  1/1    Running  0         24m
    as-release-aerospike-enterprise-2  1/1    Running  0         23m
    as-release-aerospike-enterprise-3  1/1    Running  0         22m

    ==> v1/Service
    NAME                             TYPE       CLUSTER-IP  EXTERNAL-IP  PORT(S)   AGE
    as-release-aerospike-enterprise  ClusterIP  None        <none>       3000/TCP  25m

    ==> v1/StatefulSet
    NAME                             READY  AGE
    as-release-aerospike-enterprise  4/4    25m
    ```
    ```sh
    $ kubectl get pv
    NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                STORAGECLASS     REASON   AGE
    local-pv-342b45ed   375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-2   aerospike-ssds            4m8s
    local-pv-3587dbec   375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-1   aerospike-ssds            4m8s
    local-pv-df716a06   375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-3   aerospike-ssds            4m8s
    local-pv-eaf4a027   375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-0   aerospike-ssds            4m8s
    ```
    ```sh
    Sep 16 2019 17:22:02 GMT: INFO (drv_ssd): (drv_ssd.c:1919) {test} /dev/xvdk: used-bytes 0 free-wblocks 3071936 write-q 0 write (0,0.0) defrag-q 0 defrag-read (0,0.0) defrag-write (0,0.0)
    ```


### Using Shadow device configuration

In cloud environments, the direct-attached or local SSDs (also called as ephemeral drives/volumes) does not guarantee persistence. These volumes are created along with the instance, and purged when the instance stops. The local ephemeral volumes are much faster compared to persistent disks which are network attached (for example, EBS volumes on AWS). Aerospike allows the [configuration of shadow devices](/docs/operations/configure/namespace/storage/#recipe-for-shadow-device) where all the writes are also propagated to a secondary persistent storage device.

```sh
storage-engine device{
        device /dev/sdb /dev/sdf
        device /dev/sdc /dev/sdg
        ...
}
```

The above example can be easily extended to use a shadow device configuration.

To use a shadow device configuration,

- Define a storageClass for provisioning of these 'shadow device' persistent disks. Kubernetes allows [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) (create storage volumes on-demand) using pre-created [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/).

    For this example, we will create a StorageClass `shadow` using `gce-pd` provisioner (since this setup is running on GKE) and specify the parameters as `type: pd-ssd` (volume type).

    ```sh
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: shadow
    provisioner: kubernetes.io/gce-pd
    parameters:
        type: pd-ssd
    ```

- Add another entry to `persistenceStorage` configuration section in `values.yaml` file to configure the shadow device. Note that the `storageClass` must point to `shadow` as defined in the previous step. This will automatically configure `volumeClaimTemplates` in `statefulset` definition during the deployment.
    ```sh
    ...
    ### Deployment specific configurations

    persistenceStorage:
    - devicePath: /dev/xvdk
        enabled: true
        name: data-dev
        storageClass: aerospike-ssds
        accessMode: ReadWriteOnce
        size: 375Gi
        volumeMode: Block
    - devicePath: /dev/xvdd
        enabled: true
        name: shadow-dev
        storageClass: shadow
        accessMode: ReadWriteOnce
        volumeMode: Block
        size: 375Gi
    ...
    ```

- Add `/dev/xvdd` to the `aerospike.template.conf` as a 'shadow device' in the namespace `storage-engine` configuration.

    ```sh
    ...
    namespace ${NAMESPACE} {
        replication-factor ${REPL_FACTOR}
        memory-size ${MEM_GB}G
        default-ttl ${DEFAULT_TTL}

        storage-engine device {
            device /dev/xvdk /dev/xvdd
            write-block-size 128K
        }
    }
    ...
    ```

- Install the chart as before by passing in the `aerospike.template.conf` and `values.yaml` file.
    ```sh
    helm install \
    --set dBReplicas=4 \
    --name as-release aerospike/aerospike-enterprise \
    --set-file featureKeyFilePath=/secrets/aerospike/features.conf \
    --values helm/values.yaml \
    --set-file confFilePath=helm/files/aerospike.template.conf
    ```
    ```sh
    $ kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                                  STORAGECLASS     REASON   AGE
    local-pv-342b45ed                          375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-2     aerospike-ssds            20h
    local-pv-3587dbec                          375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-1     aerospike-ssds            20h
    local-pv-df716a06                          375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-3     aerospike-ssds            20h
    local-pv-eaf4a027                          375Gi      RWO            Delete           Bound    default/data-dev-as-release-aerospike-enterprise-0     aerospike-ssds            20h
    pvc-3421ed54-d944-11e9-ac74-42010aa0014b   375Gi      RWO            Delete           Bound    default/shadow-dev-as-release-aerospike-enterprise-0   shadow                    89m
    pvc-5ca77b01-d944-11e9-ac74-42010aa0014b   375Gi      RWO            Delete           Bound    default/shadow-dev-as-release-aerospike-enterprise-1   shadow                    88m
    pvc-8b1aa7ea-d944-11e9-ac74-42010aa0014b   375Gi      RWO            Delete           Bound    default/shadow-dev-as-release-aerospike-enterprise-2   shadow                    86m
    pvc-c83cf6c0-d944-11e9-ac74-42010aa0014b   375Gi      RWO            Delete           Bound    default/shadow-dev-as-release-aerospike-enterprise-3   shadow                    85m
    ```
    ```sh
    Sep 17 2019 12:11:52 GMT: INFO (drv_ssd): (drv_ssd.c:3216) opened device /dev/xvdk: usable size 402653184000, io-min-size 4096
    Sep 17 2019 12:11:52 GMT: INFO (drv_ssd): (drv_ssd.c:3270) shadow device /dev/xvdd is compatible with main device, shadow-io-min-size 512
    ```