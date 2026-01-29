# Architecture overview

**Last Updated:** 2026-01-22

This section provides the architectural overview of the Spyre Operator.

## The Spyre Operator

Manages the lifecycle of Spyre components, deployments, and updates.

## Spyre scheduler

Provides scheduling logic to allocate Spyre devices (PF/VF modes). Ensures that workloads are placed on suitable worker nodes.

## Device plug-in

Detects Spyre hardware, generates topology and configuration files, and exposes them to pods. Handles `/dev/vfio`, PCI devices, and related mounts.

## Validation webhook

Ensures that user workloads and cluster policies are valid. Blocks faulty deployments or invalid CRs. For example, multiple Spyre requests, unsupported configurations).

## Custom Resource Definitions (CRDs)

- `SpyreClusterPolicy`: Defines a cluster-wide policy for scheduler, device plug-in, and validation settings.
- `SpyreNodeState`: Tracks Spyre devices, PCI addresses, health, and allocation state on each worker node.

## User workloads

Pods request Spyre resources that use resource limits. The operator and the scheduler handle placement and allocation.

*Figure 1. Architecture Overview*

![Spyre Operator Architectural Overview](../../assets/images/spyre-operator-overview.png)

## Parent topic:

[Introduction to the Spyre Operator](introduction.md)
