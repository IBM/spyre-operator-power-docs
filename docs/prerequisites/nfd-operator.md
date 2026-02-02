# Node Feature Discovery (NFD) Operator

**Last Updated:** 2026-01-22

The Node Feature Discovery (NFD) Operator is a Red Hat OpenShift Operator that automates the detection of hardware features and system configurations across cluster nodes.

The NFD Operator detects hardware features and configuration in an Red Hat OpenShift Cluster and labels nodes with hardware-specific information. The NFD Operator gives the Spyre node a specific label so spyre-necessary pods know which node to run on.

## Procedure

1. In the Red Hat OpenShift Web Console, click **Operators > OperatorHub**.
2. Search for **Node Feature Discovery**.
3. Click the **Node Feature Discovery** tile.
4. Click **Install**.
5. On the Install Operator page, Select the configuration. A specific namespace on the cluster

> **Note:** You do not need to create a namespace because it is already created for you.

6. Click **Install**.
7. Verify that the NFD Operator is successfully installed:
   a. Navigate to **Operators > Installed Operators**.
   b. Make sure that you select **All Projects** or **openshift-nfd** in the **Projects** drop-down list.
   c. Check for **Node Feature Discovery** with the Succeeded status.

8. If you have installed the NFD Operator is installed successfully, click **NFD Operator**.
9. Click the **Node Feature Discovery** tab.
10. Select **Create Node Feature Discovery**.
11. Apply the following YAML configuration file:

```yaml

apiVersion: nfd.openshift.io/v1
kind: NodeFeatureDiscovery
metadata:
  name: nfd-instance
  namespace: openshift-nfd
spec:
  extraLabelNs:
  - ibm.com
  instance: "" # instance is empty by default
  operand:
    image: 'registry.redhat.io/openshift4/ose-node-feature-discovery-rhel9:v4.19'
    imagePullPolicy: Always
    servicePort: 12000
  workerConfig:
    configData: |
      #core:
      #  labelWhiteList:
      #  noPublish: false
      #  sleepInterval: 60s
      #  sources: [all]
      #  klog:
      #    addDirHeader: false
      #    alsologtostderr: false
      #    logBacktraceAt:
      #    logtostderr: true
      #    skipHeaders: false
      #    stderrthreshold: 2
      #    v: 0
      #    vmodule:
      ##   NOTE: the following options are not dynamically run-time configurable
      ##         and require a nfd-worker restart to take effect after being changed
      #    logDir:
      #    logFile:
      #    logFileMaxSize: 1800
      #    skipLogHeaders: false
      #sources:
      #  cpu:
      #    cpuid:
      ##     NOTE: whitelist has priority over blacklist
      #      attributeBlacklist:
      #        - "BMI1"
      #        - "BMI2"
      #        - "CLMUL"
      #        - "CMOV"
      #        - "CX16"
      #        - "ERMS"
      #        - "F16C"
      #        - "HTT"
      #        - "LZCNT"
      #        - "MMX"
      #        - "MMXEXT"
      #        - "NX"
      #        - "POPCNT"
      #        - "RDRAND"
      #        - "RDSEED"
      #        - "RDTSCP"
      #        - "SGX"
      #        - "SSE"
      #        - "SSE2"
      #        - "SSE3"
      #        - "SSE4.1"
      #        - "SSE4.2"
      #        - "SSSE3"
      #      attributeWhitelist:
      #  kernel:
      #    kconfigFile: "/path/to/kconfig"
      #    configOpts:
      #      - "NO_HZ"
      #      - "X86"
      #      - "DMI"
      #  pci:
      #    deviceClassWhitelist:
      #      - "0200"
      #      - "03"
      #      - "12"
      #    deviceLabelFields:
      #      - "class"
      #      - "vendor"
      #      - "device"
      #      - "subsystem_vendor"
      #      - "subsystem_device"
      #  usb:
      #    deviceClassWhitelist:
      #      - "0e"
      #      - "ef"
      #      - "fe"
      #      - "ff"
      #    deviceLabelFields:
      #      - "class"
      #      - "vendor"
      #      - "device"
      #  custom:
      #    - name: "my.kernel.feature"
      #      matchOn:
      #        - loadedKMod: ["example_kmod1", "example_kmod2"]
      #    - name: "my.pci.feature"
      #      matchOn:
      #        - pciId:
      #            class: ["0200"]
      #            vendor: ["15b3"]
      #            device: ["1014", "1017"]
      #        - pciId :
      #            vendor: ["8086"]
      #            device: ["1000", "1100"]
      #    - name: "my.usb.feature"
      #      matchOn:
      #        - usbId:
      #          class: ["ff"]
      #          vendor: ["03e7"]
      #          device: ["2485"]
      #        - usbId:
      #          class: ["fe"]
      #          vendor: ["1a6e"]
      #          device: ["089a"]
      #    - name: "my.combined.feature"
      #      matchOn:
      #        - pciId:
      #            vendor: ["15b3"]
      #            device: ["1014", "1017"]
      #          loadedKMod : ["vendor_kmod1", "vendor_kmod2"]

```

> **NOTE:** in the `spec.operand.image`, the image tag *needs to match the OCP version that your cluster is running*


## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
