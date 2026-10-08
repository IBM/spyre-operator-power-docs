# Troubleshooting

*Last Updated: 2026-01-22*

This section presents a list of issues that could be encountered during the operation of the Spyre Operator.

## Known issues

### Missing /etc/aiu/senlib_config.json

- There is no pre-defined volume mounted to path `/etc/aiu`. This folder is reserved for volume mounted by the spyre device plugin only.

For example, the following volume mount must be removed:

```yaml
volumeMounts:
- mountPath: /etc/aiu
  name: config
```

### Pod is not scheduled as expected (Pending)

- Check requested resource name especially for the experimental per-device allocation pool.
- Confirm status of SpyreNodeState and node capacity/allocatable.
- Run the command to check the allocatable resources of the node

```bash
oc describe node <workernode-name> | grep Allocatable -A11
```

### Container status unknown

This state could happen when the spyre resource is in a race condition between multiple resource pools such as default pool and experimental mode's per-device allocation pool. Restarting the pod manually should put it back into a pending state until the device is released.

### ERROR client-go Failed to update lock: resource name may not be empty

When `"ERROR client-go Failed to update lock: resource name may not be empty "` is present in the operator log file and is followed by an operator restart indicates that the operator(manager) process could not properly connect to the kubernetes API server. The operator log file can be viewed using your choice of tools. The oc that can be used in this case is: `oc logs spyre-operator-XXX` -- substitute the XXX token for the pod hash code.

### Recommended actions

This is an expected behavior for any operator that is implemented using the operator runtime and its an indicator of network issues, communications between the node and the API server is interrupted and the same error is present in the log of other operators, it can safely be ignored.

If the error is persistent, validate the network connection between the node executing the operator pod and the API server is there and there are no other underlying issues.

If you have enough resources in the cluster consider increasing the operator deployment replica count to attenuate any service disruptions.

### Actions

- Use `Deployment` instead of Pod to deploy your application (recommended).
- If you choose to use Pod, manually retry the deployment after a few minutes.

### Metrics collection tools aiu-smi and Spyre operator dashboard do not show any data
Metrics collection has a known issue which is that the UID of the metrics-exporter and worker pods differ because of which the worker pod fails to write to the metrics file read by the metrics-exporter. Follow the below workaround to start the workload pod with the same UID as the metrics-exporter (which is UID 1001). With these steps, both aiu-smi and the Spyre operator dashboard will show metrics.
1. Create a new SCC that pins UID 1001
Requires cluster-admin access. To be run once per namespace.
```bash
cat <<'EOF' | oc create -f -
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: restricted-1001-uid
allowHostDirVolumePlugin: false
allowHostIPC: false
allowHostNetwork: false
allowHostPID: false
allowHostPorts: false
allowPrivilegeEscalation: false
allowPrivilegedContainer: false
allowedCapabilities:
- NET_BIND_SERVICE
defaultAddCapabilities: null
fsGroup:
  type: MustRunAs
readOnlyRootFilesystem: false
requiredDropCapabilities:
- ALL
runAsUser:
  type: MustRunAsRange
  uidRangeMin: 1001
  uidRangeMax: 1001
seLinuxContext:
  type: MustRunAs
seccompProfiles:
- runtime/default
volumes:
- configMap
- csi
- downwardAPI
- emptyDir
- ephemeral
- image
- persistentVolumeClaim
- projected
- secret
EOF
```

2. Bind the SCC to the InferenceService's service account
Requires cluster-admin access. To be run once per namespace.
```bash
# Find the service account
oc get inferenceservice <name> -n <project> \
  -o jsonpath='{.spec.predictor.serviceAccountName}'

# Bind the SCC
oc adm policy add-scc-to-user restricted-1001-uid \
  -z <serviceAccount> -n <project>
```

3. Patch the ServingRuntime to request UID 1001
Requires project admin access. To be run once per ServingRuntime.
```bash
oc patch servingruntime <name> -n <project> --type=json -p='[
  {
    "op": "add",
    "path": "/spec/containers/0/securityContext",
    "value": {
      "runAsUser": 1001,
      "runAsNonRoot": true,
      "allowPrivilegeEscalation": false,
      "capabilities": {"drop": ["ALL"]}
    }
  }
]'
```
This change takes effect for every new pod started from this runtime — no per-pod action needed.

4. After the next pod restart, verify by checking that the running container uses UID 1001:
```bash
# Check runAsUser in the running pod
oc get pod -n <project> \
  -l serving.kserve.io/inferenceservice=<name> \
  -o jsonpath='{.items[0].spec.containers[?(@.name=="kserve-container")].securityContext}'

# Confirm the SCC that was granted at admission
oc get pod -n <project> \
  -l serving.kserve.io/inferenceservice=<name> \
  -o jsonpath='{.items[0].metadata.annotations.openshift\.io/scc}'
```
Expected ouput: `runAsUser` is `1001` and the SCC annotation shows `restricted-1001-uid`.

## Parent topic:

[Spyre Operator for IBM Power User's Guide](../../README.md)
