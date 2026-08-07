# Secondary Scheduler Operator

**Last Updated:** 2026-01-22

The Secondary Scheduler Operator allows you to deploy a custom scheduler alongside the default Red Hat OpenShift scheduler.

To install the Secondary Scheduler Operator for Red Hat OpenShift, use the web console.

## Procedure

1. Log in to the Red Hat OpenShift web console.
2. Create the required namespace for the Secondary Scheduler Operator for Red Hat OpenShift:
   - Navigate to **Administration Namespaces** and click **Create Namespace**.
   - Enter `openshift-secondary-scheduler-operator` in the **Name** field and click **Create**.
3. Install the Secondary Scheduler Operator for Red Hat OpenShift:
   - Navigate to **Operators >OperatorHub**.
   - Enter **Secondary Scheduler Operator for Red Hat OpenShift** into the filter box.
   - Select the **Secondary Scheduler Operator for Red Hat OpenShift** and click **Install**.
   - On the **Install Operator** page:
      - The **Update channel** is set to **stable**, which installs the latest stable release of the Secondary Scheduler Operator for Red Hat OpenShift.
      - Select **A specific namespace on the cluster** and select **openshift-secondary-scheduler-operator** from the drop-down menu.
      - Select an **Update approval** strategy:
         - The **Automatic** strategy allows Operator Lifecycle Manager (OLM) to automatically update the Operator when a new version is available.
         - The **Manual** strategy requires a user with appropriate credentials to approve the Operator update.
      - Click **Install**.
4. Navigate to **Operators >Installed Operators**.
5. Verify that the **Secondary Scheduler Operator for Red Hat OpenShift** is listed with a **Status** of **Succeeded**.

**Note:** You may need to create the appropriate namespace for the secondary scheduler operator installation. If the namespace `openshift-secondary-scheduler-operator` is not present, you will have to create the namespace with the following command before the installation:

```bash
oc create namespace openshift-secondary-scheduler-operator
```

## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
