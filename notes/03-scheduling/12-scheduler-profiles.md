# 12 — Scheduler Profiles

The scheduler works through a fixed sequence of **extension points**. A **plugin** is a piece of logic attached to one or more of them. A **profile** is a named set of enabled and disabled plugins, so one `kube-scheduler` process can behave like several schedulers.

---

## 1. Plugins and extension points

A plugin is **Go code compiled into the `kube-scheduler` binary**, written against the scheduling framework's interfaces. It is not a separate process, not a controller and not an API call. The KubeSchedulerConfiguration file only decides which of the compiled-in plugins run, at which point, and with what weight.

![A Pod leaves the queue sorted by PrioritySort, passes filter plugins that remove nodes, score plugins that rank the remaining nodes, and DefaultBinder that binds it to the highest weighted score; if no node passes the filter, postFilter runs DefaultPreemption](./diagrams/14-scheduling-framework.svg)

| Phase | Extension points | Default plugins (examples) | What they do |
|---|---|---|---|
| Queue | `queueSort` | PrioritySort | Orders pending Pods by `spec.priority` ([10](10-priority-classes.md)) |
| Filter | `preFilter`, `filter` | NodeResourcesFit, NodeName, NodeUnschedulable, TaintToleration, NodeAffinity, NodePorts | Remove nodes that cannot run the Pod. NodeUnschedulable removes cordoned nodes |
| No fit | `postFilter` | DefaultPreemption | Preemption: runs only when every node was filtered out |
| Score | `preScore`, `score` | NodeResourcesFit, ImageLocality, TaintToleration, NodeAffinity | Rank the remaining nodes. Scores never reject a node |
| Bind | `reserve`, `permit`, `preBind`, `bind`, `postBind` | DefaultBinder | Write the Pod's node to the API server |

- One plugin can implement several points. For example, NodeResourcesFit filters on free resources and also scores on them.
- Each score plugin has a **weight**. The node with the highest weighted sum wins.

**Slide corrections:**
- The slide groups `reserve` under Scoring and `permit` under Binding. In the framework, `reserve` and `permit` end the scheduling cycle, and the binding cycle starts at `preBind`. The order on the slide is still right.
- The slide's profile YAML enables `MyCustomPluginA` twice in the diagram. The YAML correctly has A and B.

## 2. Profiles: several schedulers in one process

Since 1.18, one KubeSchedulerConfiguration can hold several profiles. Each profile answers to its own `schedulerName`, which Pods select with `spec.schedulerName` exactly as in [11](11-multiple-schedulers.md).

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: /etc/kubernetes/scheduler.conf
profiles:
- schedulerName: default-scheduler
- schedulerName: no-taints-scheduler
  plugins:
    filter:
      disabled:
      - name: TaintToleration
    score:
      disabled:
      - name: TaintToleration
- schedulerName: no-scoring-scheduler
  plugins:
    preScore:
      disabled:
      - name: '*'          # '*' = every default plugin at this point
    score:
      disabled:
      - name: '*'
```

- **Structure:** under `plugins`, write `<extensionPoint>`, then `enabled` or `disabled`, then a list of `name` entries. `multiPoint` enables or disables a plugin at every point it implements.
- **One process means one shared view of the nodes.** Separate scheduler processes ([11](11-multiple-schedulers.md)) can race and place two Pods into the same free space. Profiles cannot, and there is only one process to run.
- **The queue is shared too.** All profiles must use the same `queueSort` plugin.

## 3. Giving the kubeadm scheduler a config file

The kubeadm scheduler is a static Pod with no `--config`. Adding profiles means editing `/etc/kubernetes/manifests/kube-scheduler.yaml` ([09](09-static-pods.md)):

1. Save the config on the node, for example as `/etc/kubernetes/scheduler-config.yaml`.
2. Add `--config=/etc/kubernetes/scheduler-config.yaml` to the command, plus a `hostPath` volume and `volumeMount` for the file.
3. Set `clientConnection.kubeconfig` in the file. **`--kubeconfig` is ignored once `--config` is set**, so without it the scheduler cannot reach the API server.
4. Wait for the kubelet to restart the Pod, then check: `kubectl -n kube-system get pod -l component=kube-scheduler` and `kubectl -n kube-system logs <pod>`.

## 4. Your own logic

There are three ways to add your own scheduling logic, from most to least integrated:

| Approach | How | Language |
|---|---|---|
| Custom plugin | Implement a framework interface (for example `Filter(...)`), register it, and build your own `kube-scheduler` binary. [kubernetes-sigs/scheduler-plugins](https://github.com/kubernetes-sigs/scheduler-plugins) has examples | Go only |
| Scheduler extender | List an HTTP webhook under `extenders` in the config (`urlPrefix`, `filterVerb`, `prioritizeVerb`, `bindVerb`, `weight`). The scheduler calls it at the filter, score or bind step | Any |
| Your own scheduler | Watch Pods whose `spec.schedulerName` is your name and `spec.nodeName` is empty, pick a node, then POST a `Binding` to `pods/<name>/binding` | Any, with a client library |

---

## Exam angle

- Expect profiles in a question, if at all, as "configure a profile named X" or "make Pod Y use profile X". It is the same `schedulerName` mechanism as [11](11-multiple-schedulers.md).
- Write the config from the "Multiple profiles" example on the Scheduler Configuration page rather than from memory.
- Verify with the `SOURCE` column of `kubectl get events -o wide`, which shows the profile name, and the scheduler's logs.
- **Traps:**
  - `--config` without `clientConnection.kubeconfig`;
  - a manifest edit with no `hostPath` mount for the config file;
  - two profiles with different `queueSort` plugins (rejected);
  - `plugins` placed at the top level instead of under a profile.

## References

- [Scheduler Configuration](https://kubernetes.io/docs/reference/scheduling/config/) — "Extension points", "Scheduling plugins" (which plugin runs where), "Multiple profiles" YAML
- [Scheduling Framework](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/) — the extension points in order, scheduling cycle versus binding cycle
- [kube-scheduler Configuration (v1)](https://kubernetes.io/docs/reference/config-api/kube-scheduler-config.v1/) — every field: `clientConnection`, `profiles`, `extenders`
