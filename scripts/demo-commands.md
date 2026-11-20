# 12-minute demo command sequence

## 0. Open a playground

Use the [Killercoda Kubernetes Playground](https://killercoda.com/playgrounds/scenario/kubernetes). It provides a temporary kubeadm cluster with two nodes. Start a fresh session and wait for the cluster to be ready.

```bash
kubectl get nodes -o wide
```

The playground describes two 2GB nodes and removes the control-plane scheduling taint. Choose one node with sufficient allocatable CPU and modest existing CPU requests. Inspect candidates:

```bash
kubectl describe node <node-name>
```

Look at **Allocatable** and **Allocated resources**. The lab requests 1,400m CPU for the two low-priority Pods and 600m for the critical Pod. The chosen node must be able to run the two baseline Pods, but not all three Pods together. It should have enough room after one 700m low-priority Pod is removed.

Set the node name manually from the output (replace the example with the actual node name):

```bash
NODE="controlplane"
kubectl label node "$NODE" demo-preemption=true --overwrite
```

All demo Pods use this node label so another node cannot silently absorb the critical workload.

## 1. Create the demo resources (minutes 2–5)

From the repository root:

```bash
kubectl apply -f manifests/namespace.yaml
kubectl apply -f manifests/priority-classes.yaml
kubectl apply -f manifests/low-priority.yaml
kubectl apply -f manifests/pdb-example.yaml
```

Wait for both low-priority Pods:

```bash
kubectl wait --for=condition=Ready pod -l app=low-priority-workers -n preemption-demo --timeout=90s
kubectl get pods -n preemption-demo -o wide
kubectl get priorityclass
kubectl get pdb -n preemption-demo
```

You should see two Ready low-priority Pods on the selected node. The PDB has minAvailable: 1, so the scheduler can try to preserve one replica while preempting the other.

## 2. Show the resource pressure (minutes 5–7)

```bash
kubectl describe node "$NODE"
```

In **Allocated resources**, explain that Kubernetes scheduling uses resource requests, not instantaneous CPU usage. The two demo Pods request 1,400m CPU combined. The critical Pod requests another 600m. The scheduler should not be able to fit all three on this node.

## 3. Trigger preemption (minutes 7–10)

```bash
kubectl apply -f manifests/high-priority.yaml
kubectl get pods -n preemption-demo -o wide -w
```

Watch for the high-priority Pod to become schedulable after a low-priority Pod is terminated. Press Ctrl+C after the critical Pod is Running.

If you want to inspect the nomination while it is pending, use a second terminal:

```bash
kubectl get pod critical-request -n preemption-demo -o yaml
```

Look for status.nominatedNodeName. It is a scheduler nomination, not a capacity reservation or a guarantee that the Pod will bind to that exact node.

## 4. Inspect the evidence (minutes 10–12)

```bash
kubectl get pods -n preemption-demo -o wide
kubectl describe pod critical-request -n preemption-demo
kubectl get events -n preemption-demo --sort-by=.lastTimestamp
kubectl get pdb -n preemption-demo
```

To inspect cluster-wide scheduler events:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

Depending on the Kubernetes version and event retention, the victim Pod's event may show reason Preempted. If the victim has already disappeared, use the events and the before/after Pod lists as evidence.

Explain the PDB nuance accurately: the scheduler tries to avoid violating a PDB, but this is best effort. If no alternative can make the pending Pod schedulable, preemption can still happen even when it violates a PDB.

## 5. Cleanup (minutes 12–15)

```bash
kubectl delete namespace preemption-demo
kubectl delete priorityclass demo-low demo-critical
kubectl label node "$NODE" demo-preemption-
```

Deleting the namespace removes the Deployment, Pod, and PDB. PriorityClasses are cluster-scoped, so delete them separately. Remove the temporary node label.

## If it does not preempt

Check the troubleshooting guide before changing requests. The demo depends on actual allocatable CPU and existing requests on the selected node; do not assume every playground node has identical capacity.
