# Attach PCI cards to LPAR in HMC

**Last Updated:** 2026-01-22

Using the Power Hardware Management Console (HMC), attach the PCI devices to your LPAR

## Procedure

1. Log in to the HMC instance which manages the LPAR used by your worker node(s).
2. Navigate to the list of LPARs using the **partitions** link in the sidebar.
3. Search for your LPAR/worker node in the list of partitions.
4. Select your LPAR by clicking on the **Partition Name** in the first column.
5. On the partition detail page, navigate to **Physical I/O Adapters** in the sidebar.
6. On the Physical I/O Adapters page, click **+ Add Adaptor**.
7. From the popout, select the Spyre Adaptor(s) you wish to attach to the worker node/LPAR, and click **ok**.
8. On the partition detail page, click the **save** button

## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
