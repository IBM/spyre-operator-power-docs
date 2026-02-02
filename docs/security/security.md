# Security Advisory

**Last Updated:** 2026-01-28

## General Security Advisory

The Spyre Operator for Power does not provide additional security features to an existing OCP cluster. In order to fully ensure that your environment is secure, please refer to the Openshift Container Platform documentation on best practices to secure your cluster:

* [Authentication and authorization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/authentication_and_authorization/index)
* [Security and Compliance](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/security_and_compliance/index)

## NodeCompromise-01: Compromised Node Propagation Risk

In Red Hat OpenShift, a worker node is a highly privileged part of the cluster.

If a worker node becomes compromised—whether through:

- A privilege-escalation exploit in a container
- A kernel or driver vulnerability
- A misconfigured privileged pod, or
- Direct SSH/root access misuse

Then the attacker gains control over the node's host operating system leading to manipulation of local device data, SpyreNodeState, metrics, and possibly operator-managed components — causing cluster-wide security impact.

### Mitigation

- Harden nodes: OS patching, reduce attack surface, disable unnecessary host-level services
- Use node-level attestation and protect the SpyreNodeState updates (signed payloads or verify the source)
- Limit node-local credentials and avoid giving node agents broad cluster-wide write permissions
- Monitor host integrity (file integrity monitoring, EDR) and isolate suspicious nodes automatically
- Use network segmentation to limit node → control-plane attack surfaces

## Threat Workload-EoP-04 - Privilege Escalation or Host Compromise via Custom Workload Deployment

OpenShift clusters allow teams to deploy custom workloads—including Pods, Deployments, Jobs, and CronJobs. If not properly regulated, users may deploy workloads that:

- Run with elevated privileges (`privileged: true`, `CAP_SYS_ADMIN`)
- Mount the host filesystem (`hostPath:` to `/`, `/etc/kubernetes/`, `/var/lib/kubelet`)
- Access host namespaces (`hostNetwork`, `hostPID`, `hostIPC`)
- Bypass filesystem protections (disable `readOnlyRootFilesystem`)
- Access host devices (`/dev/*`, device plugins, VFIO)
- Run untrusted or unsigned images

Such configurations create a direct path to container escape, node compromise, and ultimately control-plane impact, especially when combined with kernel exploits or misconfigured security contexts.

### Mitigation

- Limit RBAC permissions for developers; use dedicated namespaces with restricted roles
- Deny workloads running as root or with unrestricted Linux capabilities
- Apply SELinux/AppArmor profiles and seccomp restrictions

---

**Parent topic:**
[Spyre Operator for IBM Power User's Guide](../../README.md)
