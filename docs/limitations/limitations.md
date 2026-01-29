# Limitations for the Spyre Operator

**Last Updated:** 2026-01-22

- For Power, the Spyre Operator does not support peer-to-peer communication (topologyAwareAllocation mode). Power supports Host DMA for inter-card communication.
- For Power, the Spyre Operator does not support the cardmanagement component of the Spyre Operator.
- For Power, the Spyre Operator does not support Virtual Function (VF) modes, only Physical Functions (PFs) are supported.
- Only one SpyreClusterPolicy resource should exist per cluster.

## Parent topic:

[Spyre Operator for IBM Power User's Guide](../../README.md)
