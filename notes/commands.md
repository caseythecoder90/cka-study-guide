# Commands reference

> Every `kubectl`, `kubeadm`, `etcdctl`, `crictl`, `systemctl` and shell command from the course and its labs, collected in one page to `Ctrl+F`. The CKA is hands-on — this is exam material, not supporting material. For focused per-topic files see [`commands/`](commands/).

## How to use this file

- `Ctrl+F` for the tool or resource (`etcdctl snapshot`, `kubeadm upgrade`, `crictl ps`, ...).
- Sections follow the course sections and are added as the course reaches them.

---

## 0. Exam environment

Per-topic detail: [`commands/setup.md`](commands/setup.md).

```bash
alias k >/dev/null 2>&1 || alias k=kubectl
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
kubectl config use-context <context-from-the-task>   # before every task
```

## 1. Core concepts

## 2. Scheduling

## 3. Logging and monitoring

## 4. Application lifecycle management

## 5. Cluster maintenance

## 6. Security

## 7. Storage

## 8. Networking

## 9. Design and install a cluster

## 10. Install with kubeadm

## 11. Helm

## 12. Kustomize

## 13. Troubleshooting

## 14. Other topics — JSONPath and advanced kubectl
