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

Once saved, it will be valuable to double-check that the cards were attached to the LPAR correctly, by running the following steps:

1. Create an SSH connection to your bastion node
2. From your bastion, SSH to the worker node `ssh core@<worker node name>`
3. Run `LSPCI` and confirm that the PCIe cards are present.

## Parent topic:

[Prerequisites on Red Hat OpenShift using OperatorHub](prerequisites.md)
