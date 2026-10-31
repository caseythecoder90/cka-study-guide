# Imperative commands and dry-run YAML generation

> The CKAD repo's [imperative.md](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/commands/imperative.md) covers the workload generators in full (`run`, `create deployment`, `expose`, ConfigMaps, Secrets, Jobs, CronJobs). This file adds the generators an administrator reaches for and grows with every chapter.

Pattern: generate, edit only what the flags can't express, apply, verify.

```bash
k create <resource> <name> ... $do > x.yaml
vim x.yaml
k apply -f x.yaml
k get/describe <resource> <name>
```

## Workloads (review)

```bash
k run web --image=nginx --port=80 $do
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
