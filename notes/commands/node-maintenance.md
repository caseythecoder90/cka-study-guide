# Node maintenance commands

Chapter: [06-01 OS upgrades: drain, cordon, uncordon](../06-cluster-maintenance/01-os-upgrades-drain-and-cordon.md).

## Take a node out of service

```bash
k drain node-1 --ignore-daemonsets                        # cordon + evict; respects PDBs
k drain node-1 --ignore-daemonsets --force                # also delete bare Pods (lost for good)
k drain node-1 --ignore-daemonsets --delete-emptydir-data # also evict Pods with emptyDir
k drain node-1 --ignore-daemonsets --timeout=120s         # give up instead of waiting forever on a PDB
k cordon node-1                                           # no new Pods; running ones stay
```

## Check

```bash
k get nodes                                               # Ready,SchedulingDisabled
k get pods -A -o wide --field-selector spec.nodeName=node-1
k describe node node-1 | grep -i -A2 taints               # node.kubernetes.io/unschedulable:NoSchedule
k get pdb -A                                              # what can block a drain
```

## Put it back

```bash
k uncordon node-1                                         # schedulable again; nothing moves back
```
