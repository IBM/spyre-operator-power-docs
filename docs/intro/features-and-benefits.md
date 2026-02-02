# Features and benefits

**Last Updated:** 2026-01-22

The Spyre Operator has the following key features and benefits:

## Device management and allocation

- Device plug-in: Discovers, lists, and allocates Spyre PF (Physical Function) devices to pods
  - Simple allocation: Provides arbitrary Spyre cards
  - Per-device allocation: Allocates specific cards by PCI address

> **Note:** The per-device allocation process is not recommended for general users. Only the cluster administrator can use this process for card debugging.

## Custom scheduler

- Custom Kubernetes scheduler (spyre-scheduler) for intelligent Spyre device placement
- Topology-aware scheduling for optimal performance in tensor parallel workloads
- External device reservation support

## Pod validator webhook

- Validates pod specifications requesting Spyre devices
- Ensures proper scheduler name (spyre-scheduler) is specified
- Prevents configuration errors before pod deployment
- Validates resource requests and allocation mode

## Custom resource definitions (CRDs)

- **SpyreClusterPolicy**: Centralized configuration for entire cluster setup
- **SpyreNodeState**: Per-node status tracking and resource availability

## Configuration management

- Automatic generation of device configuration files (`senlib_config.json` or `config.json`)
- Topology file (`topo.json`) generation and injection
- Resource pool metadata injection
- ConfigMap that is based on topology management

## Parent topic:

[Introduction to the Spyre Operator](introduction.md)
