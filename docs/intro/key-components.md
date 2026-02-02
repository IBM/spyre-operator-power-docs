# Key components

**Last Updated:** 2026-01-22

The Spyre operator manages the full lifecycle of Spyre related components, which includes:

## Device plug-in for Spyre

Exposes the physical Spyre cards as a resource that Kubernetes can schedule and manage.

## Automatic node labeling

Labels the nodes automatically within the Red Hat OpenShift cluster that have Spyre Accelerators installed, simplifying workload placement.

## Spyre scheduler

Provides scheduling logic to allocate Spyre devices (PF/VF modes) and ensures that workloads are placed on suitable compute nodes.

## Validation webhook

Ensures that user workloads and cluster policies are valid. Blocks faulty deployments or invalid CRs. For example, multiple Spyre requests and unsupported configurations.

## Parent topic:

[Introduction to the Spyre Operator](introduction.md)
