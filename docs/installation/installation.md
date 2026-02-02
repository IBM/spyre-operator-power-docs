# Installing the Spyre Operator

**Last Updated:** 2026-01-22

The Spyre Operator helps your cluster run AI workloads more efficiently. It improves scheduling, manages resources, works with Spyre operators to optimize spyre use, supports AI pipelines, and integrates with IBM AI services.

1. Search The `Spyre Operator` on the OperatorHub page in the Red Hat OpenShift Web Console.
2. Select the **Spyre Operator**.
3. Click the **Spyre Operator**, the **Install Operator** page appears.
4. In the **Installed Namespace** from the menu, select `spyre-operator`.
5. Click **Install**.
6. Wait for the ready tab and click **View Operator**.
7. The Operator is installed successfully, if the status is set to **Succeeded**.
8. Click **SpyreClusterPolicy** tab on the installed Operators details page.
   Example YAML:

```yaml
apiVersion: spyre.ibm.com/v1alpha1
kind: SpyreClusterPolicy
metadata:
  name: spyreclusterpolicy
spec:
  experimentalMode:
    bestDeviceAllocation
    loglevel: info
  devicePlugin:
    repository: "quay.io/ibm-aiu"
    image: "spyre-device-plugin"
    version: 1.1.1
    configPath: /etc/aiu
    configName: senlib_config.json
  initContainer:
    repository: "quay.io/ibm-aiu"
    image: "spyre-device-plugin-init"
    version: 1.1.1
    executePolicy: IfNotPresent
  scheduler:
    repository: "quay.io/ibm-aiu"
    image: "spyre-scheduler"
    version: 1.1.1
  podValidator:
    enabled: true
    repository: "quay.io/ibm-aiu"
    image: "spyre-webhook-validator"
    version: 1.1.1
  cardManagement:
    enabled: false
  metricsExporter:
    enabled: true
    repository: "quay.io/ibm-aiu"
    image: "spyre-exporter"
    version: 1.1.1
```

## Configuration Fields

| Field | Type | Description | Default |
|-------|------|-------------|---------|
| **Device plug-in Configuration** | | | |
| spec.devicePlugin.configName | string | Configuration file name inside the container | senlib_config.json |
| spec.devicePlugin.configPath | string | Configuration file path inside the container | /etc/aiu |
| spec.devicePlugin.env | [] object | Environment variables for the device plug-in container | - |
| spec.devicePlugin.topologyConfigMap | string | Name of ConfigMap containing topology configuration | - |
| **Device plug-in Init Container** | | | |
| spec.devicePlugin.initContainer.executePolicy | enum | When to run the init container: Always or IfNotPresent | IfNotPresent |
| **Pod Validator Configuration** | | | |
| spec.podValidator.enabled | boolean | Enable or disable the pod validation webhook | true |
| **Other Configuration** | | | |
| spec.cardManagement | object | Card management component configuration | Not Supported for s390x |
| spec.experimentalMode | []enum | Experimental features to enable. Options: `perDeviceAllocation` (specify devices by PCI address), `topologyAwareAllocation` (policy-based allocation), `externalDeviceReservation` (use scheduler for reservation) | - |
| spec.loglevel | enum | Operator log level: debug, info, or error | info |
| spec.metricsExporter | object | Metrics exporter component configuration | NOt supported for s390x |
| spec.skipUpdateComponents | []string | Components skip updating if already deployed. Options: commonInit, devicePlugin, cardManagement, metricsExporter, scheduler, podValidator | - |

> **Note:**
> - Only one **SpyreClusterPolicy** resource should exist per cluster.
> - Device plug-in pod (DaemonSet) is scheduled on all nodes where spyre cards are attached.

9. Once the **SpyreClusterPolicy** changes are done, if any, click **Create**.
10. Check if the **SpyreClusterPolicy** is ready.
11. Run the following command to check the status of the **SpyreNodeState** resource for your worker node:

```bash
oc describe spyrenodestate <worker-node-name>
```

12. The output displays details for each attached Spyre card, including health and PCI addresses.

```
Name:         <worker-node-name>
Namespace:    
Labels:       <none>
Annotations:  <none>
API Version:  spyre.ibm.com/v1alpha1
Kind:         SpyreNodeState
Metadata:
  Generation:         2
  Owner References:
    API Version:  spyre.ibm.com/v1alpha1
    Block Owner Deletion:  true
    Controller:
    Kind:         SpyreClusterPolicy
    Name:         spyreclusterpolicy
    UID:          29cc550a-bdd1-42b1-acfb-5b8de57e852a
  Resource Version:  16078920
  UID:               45b6d05f-240c-4f8e-b6bb-fc9672162ca4
Spec:
  Node Name:  <worker-node-name>
  Spyre SSA Interfaces:
    Health:      healthy
    Pci Address: 0001:00:00.0
    Health:      healthy
    Pci Address: 0002:00:00.0
    Health:      healthy
    Pci Address: 0003:00:00.0
    Health:      healthy
    Pci Address: 0004:00:00.0
    Health:      healthy
    Pci Address: 0005:00:00.0
    Health:      healthy
    Pci Address: 0006:00:00.0
    Health:      healthy
    Pci Address: 0007:00:00.0
    Health:      healthy
    Pci Address: 0008:00:00.0
Status:
  Allocation:
    Devices:
      0003:00:00.0
      0004:00:00.0
      0001:00:00.0
      0002:00:00.0
    Pod:
      Name:        granite-3-3-8b-instruct-predictor-7676bdfec9-5dxgr
      Namespace:   spd-instance-1
      Pool:        spyre_vf
```

13. You can also check that the cards are available on the node itself by running `oc describe nodes <your node name>`, you should expect to see output similar to what's below:

```bash
Name:               worker-1
Roles:              spyre,worker
Labels:             beta.kubernetes.io/arch=ppc64le
                    beta.kubernetes.io/os=linux
...
Capacity:
  cpu:                            32
  ephemeral-storage:              125364204Ki
  hugepages-1Gi:                  0
  hugepages-2Mi:                  0
  ibm.com/spyre_pf:               3
  ibm.com/spyre_pf_0302_60_00.0:  1
  ibm.com/spyre_pf_0303_70_00.0:  1
  ibm.com/spyre_pf_0304_80_00.0:  1
  ibm.com/spyre_pf_tier0:         3
  ibm.com/spyre_pf_tier1:         3
  ibm.com/spyre_pf_tier2:         3
  memory:                         33138944Ki
  pods:                           250
...
```

This is an example from a worker node which had 3 cards installed, in the PCIe addresses `0302_60_00.0`, `0303_70_00.0`, and `0304_80_00.0`.

> **Note:** Only create one **SpyreClusterPolicy** in the entire Red Hat OpenShift cluster.

## Deploying the Spyre Operator manually through CLI

> **Note:** This is an optional step to install the Spyre Operator in case the RHOCP OperatorHub is not used for installation. The Spyre Operator is a part of certified operators; you have to enable certified-operators, before installation.

1. Create a YAML file of kind `OperatorGroup`, that is similar to the following example:

```yaml
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: spyre-operator-group
  namespace: spyre-operator
```

2. Apply the YAML file by running the following command:

```bash
oc apply -f operator-group.yaml
```

3. Create a YAML file of kind `Subscription`, that is similar to the following example:

```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: spyre-operator
  namespace: spyre-operator
spec:
  channel: "stable"
  name: spyre-operator
  source: ibm-spyre-operators
  sourceNamespace: openshift-marketplace
```

4. Apply the YAML file by running the following command:

```bash
oc apply -f subscription.yaml
```

## Parent topic:

[Spyre Operator for IBM Power User's Guide](../../README.md)
