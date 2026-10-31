# Imperative commands and dry-run YAML generation

> Hand-typing YAML is slow and error-prone. Imperative commands create a resource in one line, and the same command with `--dry-run=client -o yaml` (`$do`, see [setup.md](setup.md)) prints a working manifest to edit instead. This is the biggest time-saver on the exam. The file grows with every chapter.

| Entry point | Creates |
|---|---|
| `kubectl run` | Pods only |
| `kubectl create <kind>` | Most other resources: Deployments, Services, ConfigMaps, Secrets, Jobs, CronJobs, RBAC objects, quotas, ... |
| `kubectl expose` | A Service for an existing Pod, Deployment, ReplicaSet or Service |

`--dry-run=client` builds the object in kubectl and sends nothing. `--dry-run=server` sends it for full validation and admission but still persists nothing — use it to check an edited file before applying.

Pattern: generate, edit only what the flags can't express, apply, verify.

```bash
k create <resource> <name> ... $do > x.yaml
vim x.yaml
k apply -f x.yaml
k get/describe <resource> <name>
```

## Workloads

```bash
k run web --image=nginx --port=80 --labels=app=web $do
k run tmp --image=busybox --rm -it --restart=Never -- sh   # throwaway debug Pod
k run sleeper --image=busybox $do -- sleep 3600            # everything after -- is the container command
k create deployment web --image=nginx --replicas=3 $do
k expose deployment web --port=80 --target-port=8080 --type=NodePort $do
k create configmap app --from-literal=K=V $do
k create secret generic app --from-literal=K=V $do
k create job once --image=busybox $do -- date
k create cronjob tick --image=busybox --schedule="*/5 * * * *" $do -- date
```

## Access control

```bash
k create serviceaccount build-bot $do
k create role pod-reader --verb=get,list,watch --resource=pods $do
k create rolebinding pod-reader-bind --role=pod-reader --user=jane --serviceaccount=dev:build-bot $do
k create clusterrole node-reader --verb=get,list --resource=nodes $do
k create clusterrolebinding node-reader-bind --clusterrole=node-reader --group=ops $do
k create token build-bot                             # short-lived token for a ServiceAccount
k auth can-i list pods --as=jane -n dev              # verify
k auth can-i list pods --as=system:serviceaccount:dev:build-bot -n dev
```

## Cluster-level policy

```bash
k create quota team-a --hard=pods=10,requests.cpu=4,requests.memory=8Gi -n team-a $do
k create priorityclass high --value=1000 --description="critical" $do
k create ingress web --rule="app.example.com/*=web:80" $do
```

Resources with **no generator** — PV, PVC, StorageClass, NetworkPolicy, LimitRange, DaemonSet, Gateway, HTTPRoute — start from the kubernetes.io page for that resource. A DaemonSet can be generated as a Deployment and edited: change `kind`, delete `replicas` and `strategy`.
