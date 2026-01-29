# cert-manager Operator for Red Hat OpenShift

**Last Updated:** 2026-01-22

The cert-manager Operator for Red Hat OpenShift is a cluster-wide service that automates application certificate lifecycle management.

## Procedure

1. Log in to the Red Hat OpenShift web console.
2. Navigate to **Operators >OperatorHub**.
3. Enter **cert-manager Operator for Red Hat OpenShift** in the filter box.
4. Select the **cert-manager Operator for Red Hat OpenShift**.
5. Select the cert-manager Operator for Red Hat OpenShift version from **Version** drop-down list, and click **Install**.
6. On the **Install Operator** page:
   a. Update the **Update channel**, if necessary. The channel defaults to **stable-v1**, which installs the latest stable release of the cert-manager Operator for Red Hat OpenShift.
   b. Choose the **Installed Namespace** for the Operator. The default Operator namespace is `cert-manager-operator`
      If the `cert-manager-operator` namespace does not exist, it is created for you

> **Note:** During the installation, the Red Hat OpenShift web console allows you to select between `AllNamespaces` and `SingleNamespace` installation modes. For installations with cert-manager Operator for Red Hat OpenShift version 1.15.0 or later, it is recommended to choose the `AllNamespaces` installation mode. `SingleNamespace` and `OwnNamespace` support will remain for earlier versions but will be deprecated in future versions.

   c. Select an **Update approval** strategy.
   d. Click **Install**
      - The **Automatic** strategy allows Operator Lifecycle Manager (OLM) to automatically update the Operator when a new version is available
      - The **Manual** strategy requires a user with appropriate credentials to approve the Operator update

7. Navigate to **Operators >Installed Operators**
8. Verify that **CertManager Operator for Red Hat OpenShift** is listed with a **Status** of **Succeeded** in the `cert-manager-operator` namespace
9. Verify that cert-manager pods are up and running by entering the following command:

```bash
$ oc get pods -n cert-manager
```

Example output:

```bash
NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-bd7fbb9fc-wvbbt               1/1     Running   0          3m39s
cert-manager-cainjector-5bcc5f9868-7g927   1/1     Running   0          4m5s
cert-manager-webhook-d4479d7f7-9dg9W       1/1     Running   0          4m9s
```

> **Note:** You can use the cert-manager Operator for Red Hat OpenShift only after cert-manager pods are up and running.

## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
