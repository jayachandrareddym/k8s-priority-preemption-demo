# Troubleshooting

## Both low-priority Pods do not become Ready

The selected node may not have enough allocatable CPU for two 700m requests plus its existing workloads.

1. Inspect `kubectl describe node "$NODE"`.
2. Read **Allocatable** and **Allocated resources**.
3. Choose a less busy node, if available.
4. Or reduce each low-priority request from `700m` to `500m` and increase the critical Pod request to `600m` only if the resulting arithmetic still creates pressure while leaving enough room after one victim is removed.

If you edit the low-priority request, reapply the Deployment and wait until both replicas are Ready before continuing.

## The critical Pod schedules immediately

The selected node has enough unrequested capacity for all three Pods, or the Pods are not constrained to the same node.

- Confirm the critical Pod has `nodeSelector: demo-preemption: "true"`.
- Confirm the low-priority Pods are on the same labelled node.
- Re-check the node's **Allocated resources**.
- Increase the critical Pod's CPU request or the baseline requests in small increments, while preserving the requirement that the baseline fits and that removing one low-priority Pod makes the critical Pod fit.

To change the request, edit `manifests/high-priority.yaml`, delete the existing critical Pod, and reapply the file.

## The critical Pod remains Pending after a low-priority Pod exits

The freed resources may still be insufficient, the node may have another scheduling constraint, or the chosen node may no longer be feasible.

- Inspect the critical Pod's Events.
- Re-check node CPU and memory allocatable/requested values.
- Ensure the high-priority Pod and baseline Pods use the same node selector.
- Confirm the high PriorityClass value is greater than the low PriorityClass value.
- Reduce the critical Pod's request or adjust the baseline request values, then recreate the Pod.

## No Preempted event is visible

Events are time-limited and exact wording varies. Use the before/after Pod list, the critical Pod's events, and the scheduler event stream as evidence. Capture the event output immediately after the demo.

## PDB behavior differs from expectation

PDB handling in scheduler preemption is best effort. The scheduler tries to choose victims without violating PDBs, but can violate them when no alternative victim set makes the pending Pod schedulable. Do not present a PDB as a hard barrier to preemption.

## Cleanup did not remove the PriorityClasses

PriorityClasses are cluster-scoped, not namespaced. Delete them explicitly:

```bash
kubectl delete priorityclass demo-low demo-critical
```
