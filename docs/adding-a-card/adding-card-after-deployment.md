# Adding a card after deployment

**Last Updated:** 2026-01-22

Once the new Spyre cards are attached and the operator is deployed, perform the following steps:

## Procedure

1. Re-run the steps in Mounting Spyre devices to worker node.
2. In the existing `SpyreClusterPolicy`, set the following value (`oc edit spyreclusterpolicy spyreclusterpolicy`)

```
spec.devicePlugin.initContainer.executePolicy: Always
```

3. Remove the existing `SpyreClusterPolicy` and redeploy it with the updated value provided below:

```
spec.devicePlugin.initContainer.executePolicy: IfNotPresent
```

## Parent topic:

[Spyre Operator for IBM Power User's Guide](../../README.md)
