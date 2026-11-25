# 12 — In-Place Pod Resize

Change the CPU and memory of a **running** Pod without recreating it. The Pod keeps its name, UID and IP, and the container may not even restart.

| Version | Stage |
|---|---|
| 1.27 | Alpha, off by default (what the course video shows) |
| 1.33 | Beta, **on by default** |
| **1.35** | **Stable.** Always on, so no feature gate to set |

Runnable version: [`examples/resize/resize-demo.yaml`](../../examples/resize/resize-demo.yaml). Setting requests and limits in general: [03-07](../03-scheduling/07-resource-requirements.md).

---

## 1. Resizing

It goes through the Pod's **`resize` subresource**. A plain `kubectl edit pod` is still refused.

```bash
k edit pod resize-demo --subresource resize          # easiest: edit the resources, save
k patch pod resize-demo --subresource resize -p \
  '{"spec":{"containers":[{"name":"app","resources":{"requests":{"cpu":"800m"},"limits":{"cpu":"800m"}}}]}}'
```

**Editing a Deployment still replaces its Pods.** It changes the template, so it starts a rollout, even on 1.35+. In-place resize applies to a Pod. A resized Pod of a Deployment keeps its new size only until it's replaced, because the template still has the old values.

## 2. Does the container restart?

Set per resource by `resizePolicy`:

```yaml
  containers:
  - name: app
    resizePolicy:
    - resourceName: cpu
      restartPolicy: NotRequired         # default
    - resourceName: memory
      restartPolicy: RestartContainer    # restart the container (not the Pod) to apply
```

Use `RestartContainer` for memory when the app sizes itself at startup. A JVM with a heap fixed at start won't use extra memory until it restarts.

## 3. Did it work?

`spec` is what you **asked for**. `status` is what the container **actually has**:

```bash
k get pod resize-demo -o jsonpath='{.spec.containers[0].resources}{"\n"}{.status.containerStatuses[0].resources}{"\n"}'
k describe pod resize-demo               # conditions and events
```

| Condition (reason) | Meaning | What to do |
|---|---|---|
| none, and `status` matches `spec` | Done | |
| `PodResizePending` (`Deferred`) | Doesn't fit on the node **right now** | Wait, or free up the node. The kubelet retries |
| `PodResizePending` (`Infeasible`) | Can **never** fit on this node | Ask for less, or recreate the Pod elsewhere |
| `PodResizeInProgress` | Accepted, being applied | Usually brief. If stuck, read its message |

A resize never moves the Pod to another node.

### 3.1 Lowering a memory limit below current usage

**You can lower a memory limit.** The catch is lowering it below what the container is using **at that moment**. Memory in use can't be taken away from a process, and a limit below usage would get it OOM-killed. With `NotRequired`, the kubelet therefore **skips the change and waits**:

1. The limit is 512Mi, and the app is using 400Mi.
2. You resize the limit to 300Mi. `spec` says 300Mi.
3. 400Mi > 300Mi, so the kubelet leaves the real limit at 512Mi. `status` says 512Mi, and the condition is `PodResizeInProgress`.
4. The app frees memory and drops to 250Mi. Now 300Mi is **feasible**, so the kubelet applies it. The condition clears.

"Until the desired memory limit is feasible" means step 4: until usage falls below the new limit. **If usage never falls, it stays in progress indefinitely.** Nothing breaks, and the old limit stays in force. To end it, raise the limit again, or restart the app.

The check is best effort: if usage spikes just after it, the container can still be OOM-killed. With `RestartContainer` for memory, the container is restarted to apply the new limit instead.

## 4. Limitations

- Only CPU and memory.
- **The QoS class can't change.** A Guaranteed Pod must keep requests equal to limits, so change both. A BestEffort Pod can't be given requests.
- Requests and limits can be changed but not removed.
- Ephemeral containers can't be resized. **Sidecars can**, and regular init containers can while they are running (the slide says init containers can't).
- Not for Windows Pods, or Pods under the static CPU or Memory manager policy.

---

## Exam angle

- **"Change Pod X's CPU without recreating it":** `k edit pod X --subresource resize`. Verify with `status.containerStatuses[].resources` and an unchanged restart count.
- **"Change the resources of Deployment X":** `k set resources deploy X`. Expect new Pods.
- **`edit` says the field is immutable:** you left out `--subresource resize`.
- **Traps:** a request above its limit is rejected (the slide has `cpu: "1"` requested with a `500m` limit); changing only one of the two on a Guaranteed Pod.

## References

- [Resize CPU and memory resources assigned to containers](https://kubernetes.io/docs/tasks/configure-pod-container/resize-container-resources/) — "Container resize policies" (YAML), "Example 1" (the patch command), "Pod resize status", "Limitations"
- [In-place Pod resize is GA in 1.35](https://kubernetes.io/blog/2025/12/19/kubernetes-v1-35-in-place-pod-resize-ga/) — what changed, including memory limit decreases (blog: not open during the exam)
