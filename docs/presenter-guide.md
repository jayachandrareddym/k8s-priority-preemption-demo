# Presenter guide — 10 to 15 minutes

## Core message

**Priority is a scheduling preference with preemption behavior, not a resource reservation.** A high-priority Pod must still fit on a node after the scheduler evaluates which lower-priority Pods could be removed.

## Timing and speaking cues

### 0:00–2:00 — Establish the scene
Say: “We have a node with finite CPU capacity. Two ordinary workloads are already running. Now a critical workload arrives. What should Kubernetes do?”

Show `kubectl get nodes -o wide` and the selected node's allocatable/allocated resources.

### 2:00–5:00 — Deploy the baseline
Apply namespace, PriorityClasses, low-priority Deployment, and PDB.

Say: “These Pods request CPU but do almost no CPU work. The scheduler accounts for requests when deciding whether a Pod fits. This lets us demonstrate scheduling pressure without running CPU stress tools.”

Show both low-priority Pods Ready and their `priorityClassName`.

### 5:00–7:00 — Make the arithmetic visible
Two low-priority Pods request 700m each: 1,400m total. The incoming critical Pod requests 600m. Together they need 2,000m, before accounting for other Pods already assigned to the node.

Say: “The low-priority Pods fit, but all three Pods do not. The scheduler cannot solve this by ignoring the requests.”

### 7:00–10:00 — Trigger the preemption
Apply `high-priority.yaml` and watch Pods.

Say: “The critical Pod is pending because the node cannot fit it. Its priority is higher, so the scheduler evaluates whether removing lower-priority Pods would make the node feasible.”

Point out the disappearing low-priority replica and the critical Pod becoming Running.

### 10:00–12:00 — Show evidence and correct the PDB myth
Show Pod list, events, PDB, and `kubectl describe pod`.

Say: “A PDB is not an absolute shield against scheduler preemption. Kubernetes tries to respect PDBs when choosing victims, but that protection is best effort. If no alternative makes the pending Pod schedulable, preemption can still violate the PDB.”

### 12:00–15:00 — Close with the production lesson

Say: “PriorityClass is platform governance. Define who can use each class, what workload qualifies, and how resource requests are reviewed. Otherwise, priority inflation turns every workload into a critical workload.”

Point out the disappearing low-priority replica and the critical Pod becoming Running.

Ask the audience: “Who owns your PriorityClasses today?”

## Technical reminders

- Priority changes scheduling queue order and can enable preemption for an unschedulable higher-priority Pod.
- The scheduler can nominate a node; `nominateinodeName` is not a reservation.
- Victims receive their graceful termination period. This is why nomination and actual scheduling may not happen at the same instant.
- PDB respect during scheduler preemption is best effort, not guaranteed.
- Priority and QoS are different concepts.
- This demo uses requested CPU, not CPU utilization, to create scheduler resource pressure.
