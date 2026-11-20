# Expected results and evidence

Exact messages and timing can differ by Kubernetes release and playground state. Use the state transitions as the primary evidence rather than depending on one exact event string.

## Before the critical Pod

- Namespace `preemption-demo` exists.
- PriorityClasses `demo-low` and `demo-critical` exist.
- Two Pods from Deployment `low-priority-workers` are Ready on the labelled node.
- PDB `low-priority-workers` reports `minAvailable: 1` and normally one allowed disruption when both replicas are healthy.

## Immediately after creating the critical Pod

- The critical Pod may briefly show Pending.
- Its events should explain why scheduling cannot proceed, commonly an insufficient CPU condition.
- The scheduler may set `status.nominatedNodeName` while it prepares to preempt victims.

## After preemption

- A lower-priority Pod may terminate; the PDB should be considered on a best-effort basis.
- The high-priority Pod should become Running if the freed resources make the node feasible.
- The victim's events may include reason `Preempted`; event retention and exact wording vary.
- The remaining low-priority Pod should remain Ready in the intended one-victim scenario.

## What to show on screen

```bash
kubectl get pods -n preemption-demo -o wide
kubectl describe pod critical-request -n preemption-demo
kubectl get events -n preemption-demo --sort-by=.lastTimestamp
kubectl get pdb -n preemption-demo
kubectl describe node "$NODE"
```

Capture the before/after Pod list if preparing backup screenshots for a live event.
