---
title: Deploying Aerospike clusters in Kubernetes
description: |
  Follow these easy steps to achieve a successful deployment of the Aerospike Database on Kubernetes
scripts:
  - /assets/scripts/utils/target_blank.js

---

Kubernetes is one of the most popular container orchestration platforms of today.
It allows you to run your containerized applications across hardware and clouds,
as well as providing a comprehensive orchestration tool for your containers.

Working with Aerospike on Kubernetes is quick and easy with Aerospike's Kubernetes manifests, which can be found at:

  - [aerospike/aerospike-kubernetes](https://github.com/aerospike/aerospike-kubernetes.git)
  - [aerospike/aerospike-kubernetes-enterprise](https://github.com/aerospike/aerospike-kubernetes-enterprise.git)

These repositories also contain [helm charts](https://github.com/aerospike/aerospike-kubernetes-enterprise/tree/master/helm),
and [different aerospike deployment configurations](https://github.com/aerospike/aerospike-kubernetes-enterprise/tree/master/examples) to get started.

These manifests and helm charts will allow you to implement a dynamically scalable Aerospike cluster using Kubernetes StatefulSets.

# Usage

1. Clone the github repository onto a machine with `kubectl` configured for
your kubernetes cluster.
```sh
git clone https://github.com/aerospike/aerospike-kubernetes-enterprise.git
cd aerospike-kubernetes-enterprise
```

2. Set your parameters.
eg:
```sh
export APP_NAME=aerospike
export NAMESPACE=default
export AEROSPIKE_NODES=3
export AEROSPIKE_FEATURE_KEY_FILE=/etc/aerospike/features.conf
...
```
You can follow the below steps or run `start.sh` script available within the repository.

3. Expand the manifest template
```sh
cat manifests/* | envsubst > expanded.yaml
```

4. Create and apply the ConfigMap:
```sh
kubectl create configmap aerospike-conf -n $NAMESPACE --from-file configs/
```
Note: To apply feature-key-file, simply add the file to `configs/` directory and create the `ConfigMap`.<br /> 
If using mounted volumes to apply the feature-key-file, you can use `AEROSPIKE_FEATURE_KEY_FILE` to specify the file path within the container.

5. Deploy:
```sh
kubectl create -f expanded.yaml
```

Detailed instructions are contained in the Github repo's [README](https://github.com/aerospike/aerospike-kubernetes-enterprise/blob/master/README.md)


## Using Helm Charts

Aerospike Helm Charts are available on Helm Hub:

- Community Edition : [aerospike/aerospike](https://hub.helm.sh/charts/aerospike/aerospike)
- Enterprise Edition : [aerospike/aerospike-enterprise](https://hub.helm.sh/charts/aerospike/aerospike-enterprise)

### Steps

1. Add Aerospike Helm repository,
```sh
helm repo add aerospike https://aerospike.github.io/aerospike-kubernetes-enterprise
```

2. Install the chart,
    - You can set the configuration values defined [here](https://github.com/aerospike/aerospike-kubernetes-enterprise/tree/master/helm#configuration) using `--set` option or provide a own custom `values.yaml` file during `helm install`.
    
      Note that the namespace related configurations (`aerospikeNamespace`, `aerospikeNamespaceMemoryGB`, `aerospikeReplicationFactor` and `aerospikeDefaultTTL`) in `values.yaml` file is intended for default single namespace configuration. If using multiple namespaces, these config items can be ignored and a separate `aerospike.conf` file or template can be used.

    - To apply your own aerospike configuration, you can set `confFilePath` to point to your own custom `aerospike.conf` file or template. Note that `confFilePath` should be a path on your machine where helm client is running. The supplied `aerospike.conf` file or template must contain a `# mesh-seed-placeholder` in `heartbeat` configuration to populate mesh configuration during peer discovery. For example,
      ```sh
      ....
	      heartbeat {
            address any
            mode mesh
            port 3002

            # mesh-seed-placeholder

            interval 150
            timeout 10
        }
      .....
      ```

    - To supply feature-key-file during the deployment **(EE only)**, use `featureKeyFilePath` to point to your `features.conf` licence file during `helm install`. Note that `featureKeyFilePath` should be a path on your machine where helm client is running.

    - For storage configuration, you can configure multiple volume mounts (filesystem type) or device mounts (raw block device) or both in `values.yaml`. Please check the default [`values.yaml` file](https://github.com/aerospike/aerospike-kubernetes-enterprise/blob/master/helm/values.yaml) for details on configuration. 

    For enterprise edition,
    ```sh
    helm install \
    --set dBReplicas=5 \
    --name as-release aerospike/aerospike-enterprise \
    --set-file featureKeyFilePath=/secrets/aerospike/features.conf \
    --values /tmp/helm/values.yaml \
    --set-file confFilePath=/tmp/aerospike_templates/aerospike.template.conf
    ```

    For community edition,
    ```sh
    helm install \
    --set dBReplicas=5 \
    --name as-release aerospike/aerospike \
    --set-file confFilePath=/tmp/aerospike_templates/aerospike.template.conf
    ```

# Storage

Storage in Kubernetes is under constant development. With regards to databases,
persistent storage is handled via StatefulSets and PersistentVolumes.

Dynamic provisioning for local devices are not supported yet. However, a local volume provisioner can be deployed to automate the provisioning of local devices.

An example Aerospike cluster deployment using local volume static provisioner can be found in the [examples](/docs/deploy_guides/kubernetes/examples) section.


### Local Persistent Volumes

Local Persistent Volume is useful for utilizing the local SSD devices
that can sometimes be found on popular cloud VM instances.

As a side benefit, the local SSDs provided are often times much faster than
network based storage (eg: AWS EBS).

{{#note}}
- Dynamic provisioning for local volumes are not supported yet.<br />
- That means you cannot create local PVs through the StatefulSet's VolumeClaimTemplates,
and must create each PersistentVolume and PersistentVolumeClaim manually.<br />
{{/note}}

{{#note}}
- Specifying `nodeAffinity` is required for local volumes.
{{/note}}

{{#note}}
- Data stored on local SSDs are ephemeral.<br /> 
- A Pod that writes to a local SSD might lose access to the data stored on the disk if the Pod is rescheduled away from that node.
{{/note}}

Examples for local Persistent Volumes usage with Aerospike can be found [here](https://github.com/aerospike/aerospike-kubernetes-enterprise/tree/master/examples)

To read up more on local volumes, along with guides on how to provision them, refer to the 
[Local Volume Static Provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner) git repo. Local volume static provisioner turned to beta in Kubernetes 1.12 and GA in 1.14.

### Raw Block Volume

[Raw block volume support](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#raw-block-volume-support) - 
Raw block access is preferred by Aerospike Server for best performance
characteristics (eg: latency stability, IOPS).

{{#note}}
Raw Block Volumes can be used in conjunction with local volumes, AWS EBS, GCP
PD, and Azure Disk volumes.
{{/note}}

Examples for Raw Block Volumes usage with Aerospike can be found [here](https://github.com/aerospike/aerospike-kubernetes-enterprise/tree/master/examples)