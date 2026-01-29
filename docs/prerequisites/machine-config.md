# Configure the MachineConfig

**Last Updated:** 2026-01-22

The applied machine configuration performs a set up on the compute node where cards are attached.

> **Note:** If you apply a MachineConfig to the **worker** role it is applied to all compute nodes, and each affected node is rebooted, including the compute node with the Spyre card attached. To avoid impacting workloads on other compute nodes, assign a dedicated role to the compute node with the Spyre card attached. Create a separate MachineConfigPool (MCP) for that role, and set the `machineconfiguration.openshift.io/role` label in your MachineConfig. This ensures that configuration changes and reboots apply only to the compute node with the Spyre card attached.

## Label the compute node

Label the compute nodes with attached spyre cards by using the spyre role. This identifies the nodes as spyre enabled and ensures that the following MachineConfigs are applied only to the attached compute nodes with spyre cards.

1. Apply the YAML by running the following command:

```bash
oc label node <node-name> node-role.kubernetes.io/spyre=''
```

> **Note:** Replace the `<node-name>` with the actual name of your compute node.

2. Verify whether the label is applied correctly by running the following command:

```bash
oc get nodes
```

3. Check for the output status:

```
NAME                      STATUS   ROLES          AGE     VERSION
bootstrap-0.ocp-ai.zwm.pok   Ready    spyre,worker   3h47m   v1.32.3
```

The node shows both **spyre** and **worker** in the ROLES column.

4. Create a YAML named mcp.yaml:

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  name: spyre
spec:
  machineConfigSelector:
    matchExpressions:
      - key: machineconfiguration.openshift.io/role
        operator: In
        values:
          - worker
          - spyre
  nodeSelector:
    matchLabels:
      node-role.kubernetes.io/spyre: ''
```

5. Apply the MachineConfigPool to your cluster by running the following command:

```bash
oc apply -f mcp.yaml
```

The following output appears after creation:

```
machineconfigpool.machineconfiguration.openshift.io/spyre created.
```

6. Verify the MachineConfigPool status by running the following command:

```bash
oc get mcp
```

7. The expected output is as follows:

```
NAME     CONFIG                                             UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
master   rendered-master-d5c5205aa0a5425f2737a1b643e2871e   True      False      False      3              3                   3                     0                      4h
spyre    rendered-spyre-c8d9c399d4c492d42c3b131f6aa48ba0    True      False      False      1              1                   1                     0                      4m
worker   rendered-worker-c8d9c399d4c492d42c3b131f6aa48ba0   True      False      False      3              3                   3                     0                      4h
```

8. The **spyre** MachineConfigPool should show:
   - UPDATED: True
   - UPDATING: False
   - DEGRADED: False
   - MACHINECOUNT matching the number of Spyre-labeled nodes

## Apply the MachineConfigs

1. Unbind the PCI devices from their default drivers.
2. Bind the devices to the VFIO driver.
3. To complete the pass through setup, create a YAML named `99-vhostuser-bind.yaml` for the required symbolic links under `/sys/bus/pci/devices`:

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: spyre
  name: 99-vhostuser-bind
spec:
  config:
    ignition:
      version: 3.4.0
    systemd:
      units:
      - name: vhostuser-bind.service
        enabled: true
        contents: |
          [Unit]
          Description=Vhostuser Interface vfio-pci Bind
          Wants=network-online.target
          After=network-online.target ignition-firstboot-complete.service
          ConditionPathExists=/etc/modprobe.d/vfio.conf
          [Service]
          Type=oneshot
          TimeoutSec=900
          ExecStart=/usr/local/bin/vhostuser
          [Install]
          WantedBy=multi-user.target
    storage:
      files:
      - contents:
          inline: vfio-pci
        filesystem: root
        mode: 0644
        path: /etc/modules-load.d/vfio-pci.conf
      - contents:
          source: data:text/plain;charset=utf-8;base64,IyEvYmluL2Jhc2gKc2V0IC1lCgpQQ0lfREVWSUNFUz0kKGxzcGNpIC1uIC1kIDEwMTQ6MDZhNyB8IGN1dCAtZCAiICIgLWYxIHwgcGFzdGUgLXNkICIsIiAtKQoKZWNobyAiJFBDSV9ERVZJQ0VTIgpJRlM9JywnIHJlYWQgLXJhIERFVklDRVMgPDw8ICIkUENJX0RFVklDRVMiCgpmb3IgVkZJT0RFVklDRSBpbiAiJHtERVZJQ0VTW0BdfSI7IGRvCiAgICBjZCAvc3lzL2J1cy9wY2kvZGV2aWNlcy8iJHtWRklPREVWSUNFfSIgfHwgY29udGludWUKCiAgICBpZiBbICEgLWYgImRyaXZlci91bmJpbmQiIF07IHRoZW4KICAgICAgICBlY2hvICJGaWxlIGRyaXZlci91bmJpbmQgbm90IGZvdW5kIGZvciAke1ZGSU9ERVZJQ0V9IgogICAgICAgIGV4aXQgMQogICAgZmkKCiAgICBpZiAhIGVjaG8gLW4gInZmaW8tcGNpIiA+IGRyaXZlcl9vdmVycmlkZTsgdGhlbgogICAgICAgIGVjaG8gIkNvdWxkIG5vdCB3cml0ZSB2ZmlvLXBjaSB0byBkcml2ZXJfb3ZlcnJpZGUiCiAgICAgICAgZXhpdCAxCiAgICBmaQoKICAgIGlmICEgWyAtZiBkcml2ZXIvdW5iaW5kIF0gJiYgZWNobyAtbiAiJFZGSU9ERVZJQ0UiID4gZHJpdmVyL3VuYmluZDsgdGhlbgogICAgICAgIGVjaG8gIkNvdWxkIG5vdCB3cml0ZSB0aGUgVkZJT0RFVklDRTogJHtWRklPREVWSUNFfSB0byBkcml2ZXIvdW5iaW5kIgogICAgICAgIGV4aXQgMQogICAgZmkKCmRvbmUK
        filesystem: root
        mode: 0744
        path: /usr/local/bin/vhostuser

```

4. Apply the YAML by running the following command:

```bash
oc apply -f 99-vhostuser-bind.yaml
```

5. Create a YAML named `99-worker-vfiopci.machineconfig`:

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    kubernetes.io/arch: ppc64le
    machineconfiguration.openshift.io/role: spyre
  name: 99-worker-vfiopci
spec:
  config:
    ignition:
      version: 3.4.0
    storage:
      files:
      - contents:
          compression: ""
          source: data:,options%20vfio-pci%20ids%3D1014%3A06a7%2C1014%3A06a8%0Aoptions%20vfio-pci%20disable_idle_d3%3Dyes%0A
        mode: 420
        path: /etc/modprobe.d/vfio-pci.conf
      - contents:
          compression: ""
          source: data:,vfio-pci%0Avfio_iommu_spapr_tce%0A
        mode: 420
        path: /etc/modules-load.d/vfio-pci.conf
      - contents:
          compression: ""
          source: data:;base64,W2NyaW8ucnVudGltZV0KZGVmYXVsdF91bGltaXRzID0gWwogICJtZW1sb2NrPS0xOi0xIgpdCg==
        mode: 420
        path: /etc/crio/crio.conf.d/10-custom
      - contents:
          compression: ""
          source: data:,SUBSYSTEM%3D%3D%22vfio%22%2C%20MODE%3D%220666%22%0A
        mode: 420
        path: /etc/udev/rules.d/90-vfio-3.rules
      - contents:
          compression: ""
          source: data:,%40sentient%20-%20memlock%20134217728%0A
        mode: 420
        path: /etc/security/limits.d/memlock.conf

```

## Apply SELinux Policy (for Spyre Operator version above 1.1.0)

You must apply the SELinux policy MachineConfig for the device plug-in to work properly from the root of the aiu-operator repository.

1. Create a YAML named `50-spyre-device-plugin-selinux-minimal`:

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  labels:
    machineconfiguration.openshift.io/role: spyre
  name: 50-spyre-device-plugin-selinux-minimal
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - contents:
          source: data:text/plain;charset=utf-8;base64,bW9kdWxlIHNweXJlX2RldmljZV9wbHVnaW5fbWluaW1hbCAxLjA7CgpyZXF1aXJlIHsKICAgIHR5cGUgY29udGFpbmVyX3Q7CiAgICB0eXBlIGNvbnRhaW5lcl9ydW50aW1lX3Q7CiAgICB0eXBlIGNvbnRhaW5lcl92YXJfcnVuX3Q7CiAgICBjbGFzcyB1bml4X3N0cmVhbV9zb2NrZXQgY29ubmVjdHRvOwogICAgY2xhc3Mgc29ja19maWxlIHdyaXRlOwp9CgojIEdyYW50IE9OTFkgdGhlIHNwZWNpZmljIHBlcm1pc3Npb25zIG5lZWRlZCBmb3IgQ1JJLU8gY29tbXVuaWNhdGlvbgphbGxvdyBjb250YWluZXJfdCBjb250YWluZXJfcnVudGltZV90OnVuaXhfc3RyZWFtX3NvY2tldCBjb25uZWN0dG87CmFsbG93IGNvbnRhaW5lcl90IGNvbnRhaW5lcl92YXJfcnVuX3Q6c29ja19maWxlIHdyaXRlOwo=
        mode: 0644
        path: /etc/selinux/spyre_device_plugin_minimal.te
    systemd:
      units:
      - contents: |
          [Unit]
          Description=Install minimal SELinux policy for spyre device plugin
          After=multi-user.target

          [Service]
          Type=oneshot
          ExecStartPre=/bin/bash -c 'if [ ! -f /etc/selinux/spyre_device_plugin_minimal.pp ]; then checkmodule -M -m -o /etc/selinux/spyre_device_plugin_minimal.mod /etc/selinux/spyre_device_plugin_minimal.te && semodule_package -o /etc/selinux/spyre_device_plugin_minimal.pp -m /etc/selinux/spyre_device_plugin_minimal.mod; fi'
          ExecStart=/bin/bash -c 'semodule -i /etc/selinux/spyre_device_plugin_minimal.pp || true'
          RemainAfterExit=true

          [Install]
          WantedBy=multi-user.target
        enabled: true
        name: install-spyre-selinux-minimal-policy.service
      - contents: |
          [Unit]
          Description=Setup device plugin directories with permissions and SELinux context
          After=network-online.target
          Before=kubelet.service

          [Service]
          Type=oneshot
          # Fix kubelet directory permissions for device plugin socket operations
          ExecStart=/usr/bin/chmod 770 /var/lib/kubelet/plugins_registry
          ExecStart=/usr/bin/chmod 770 /var/lib/kubelet/device-plugins

          # delete device plugin directories before creating it
          ExecStart=/usr/bin/rm -f /usr/local/etc/device-plugins/complete
          ExecStart=/usr/bin/rm -rf /usr/local/etc/device-plugins/spyre-config
          ExecStart=/usr/bin/rm -rf /usr/local/etc/device-plugins/spyre-metrics
          ExecStart=/usr/bin/rm -rf /usr/local/etc/device-plugins/metadata

          # Create device plugin directories
          ExecStart=/usr/bin/mkdir -p /usr/local/etc/device-plugins/spyre-config
          ExecStart=/usr/bin/mkdir -p /usr/local/etc/device-plugins/spyre-metrics
          ExecStart=/usr/bin/mkdir -p /usr/local/etc/device-plugins/metadata

          # Set permissions for group write access (device plugin runs as UID 1001, GID 0)
          ExecStart=/usr/bin/chmod 770 /usr/local/etc/device-plugins
          ExecStart=/usr/bin/chmod 770 /usr/local/etc/device-plugins/spyre-config
          ExecStart=/usr/bin/chmod 770 /usr/local/etc/device-plugins/spyre-metrics
          ExecStart=/usr/bin/chmod 770 /usr/local/etc/device-plugins/metadata

          # Fix SELinux context for container access
          ExecStart=/usr/bin/chcon -R -t container_file_t /usr/local/etc/device-plugins
          ExecStart=/usr/bin/chcon -R -t container_file_t /usr/local/etc/device-plugins/spyre-config
          ExecStart=/usr/bin/chcon -R -t container_file_t /usr/local/etc/device-plugins/spyre-metrics
          ExecStart=/usr/bin/chcon -R -t container_file_t /usr/local/etc/device-plugins/metadata
          RemainAfterExit=true

          [Install]
          WantedBy=multi-user.target
        enabled: true
        name: setup-device-plugin-directories.service

```

2. Apply the YAML by running following command:

```bash
oc apply -f 50-spyre-device-plugin-selinux-minimal.yaml
```

Once the machine configs are applied, monitor the status by running following command,ensure that the UPDATED value is True and UPDATING value is False.

```bash
# oc get mcp worker
```

> **Note:**
> - When applying machine config compute nodes become unschedulable.
> - You can taint the compute node by running the following command:
>
> ```bash
> oc adm taint nodes <node-name> ibm.com/spyre=:NoSchedule
> ```

## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
