# Kubernetes Priority & Preemption — Live Demo

A rehearsed 10–15 minute Kubernetes lab for the talk **“Surviving Resource Starvation – Pod Priority & Preemption in Production Kubernetes.”**

The lab demonstrates scheduler preemption on a deliberately constrained single-node playground. It is designed for a temporary browser-based Kubernetes cluster such as Killercoda.

> **Important:** This is a controlled teaching exercise, not a production configuration. Preemption only works when the scheduler can find a node where removing lower-priority Pods makes the higher-priority Pod schedulable. Exact victim selection and event wording can vary by Kubernetes version and cluster state.

## Repository layout

- `manifests/priority-classes.yaml` — demo PriorityClasses
- `manifests/low-priority.yaml` — low-priority workloads that consume requested CPU
- `manifests/high-priority.yaml` — critical workload used to trigger preemption
- `manifests/pdb-example.yaml` — optional PDB discussion (not a shield against scheduler preemption)
- `scripts/demo-commands.md` — exact command sequence
- `docs/presenter-guide.md` — timing and speaking cues
- `docs/troubleshooting.md` — failure diagnosis and recovery
- `docs/expected-results.md` — evidence to look for

## Prerequisites

- A disposable Kubernetes playground with `kubectl` access
- At least one schedulable worker node
- Permission to create namespaces, PriorityClasses, Pods and PDBs
- A cluster with enough allocatable CPU for the baseline Pods, but not enough for the critical Pod until one or more lower-priority Pods are preempted

## Quick start

Open [Killercoda Kubernetes Playground](https://killercoda.com/playgrounds/scenario/kubernetes), wait for the cluster to become ready, then follow [scripts/demo-commands.md](scripts/demo-commands.md).

The manifests use explicit CPU requests because the scheduler uses resource requests for placement decisions. They do not rely on actual CPU consumption or stress generators.

## What the demo proves

1. Priority affects scheduling order and can enable preemption when a higher-priority Pod is unschedulable due to insufficient resources.
2. Priority is not a reservation: a high-priority Pod still needs a feasible node.
3. The scheduler may nominate a node and begin terminating selected victims; graceful termination can delay the new Pod becoming Running.
4. A PodDisruptionBudget does **not** prevent scheduler preemption. PDBs constrain voluntary disruptions through the Eviction API; scheduler preemption is a separate mechanism.
5. Priority is not QoS, and it does not protect a Pod from every form of node pressure or OOM termination.

## Cleanup

Run the cleanup commands at the end of the runbook. The lab creates a dedicated namespace and two cluster-scoped PriorityClasses named `demo-low` and `demo-critical`. Delete those resources only after ensuring they are not used by another workload.

## Further reading

- [Pod Priority and Preemption — Kubernetes documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)
- [Pod Disruption Budgets — Kubernetes documentation](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/)
- [Assigning Pods to Nodes / resource requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

## Demo status

Manifests and flow are prepared for rehearsal. **Do not call the demo verified on your target playground until you run it there and confirm that the high-priority Pod is scheduled after preemption.** Node capacity and pre-existing system workloads differ between playground sessions.
