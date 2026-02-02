# Troubleshooting

**Last Updated:** 2026-01-22

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

## Parent topic:

[Spyre Operator for IBM Power User's Guide](../../README.md)
