# Exam environment setup

The first two minutes of the exam. The PSI desktop usually ships with `k` aliased and bash completion on — check before retyping, and add only what is missing.

```bash
alias k >/dev/null 2>&1 || alias k=kubectl
complete -o default -F __start_kubectl k       # completion for the alias
export do="--dry-run=client -o yaml"            # k run web --image=nginx $do > web.yaml
export now="--force --grace-period=0"           # k delete pod web $now
```

```vim
" ~/.vimrc — YAML-friendly indentation
set expandtab tabstop=2 shiftwidth=2
```

## Before every task

```bash
kubectl config use-context <context-from-the-task>    # copy it from the task, every time
kubectl config current-context
kubectl config set-context --current --namespace=<ns> # only if the task is namespace-heavy
```

## Getting onto a node

```bash
ssh <node>              # the task names the node
sudo -i                 # most node tasks need root
exit                    # back to the base terminal before the next task — contexts live there
```

Vim keystrokes and the imperative generators carry over from the CKAD repo: [vim.md](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/commands/vim.md) · [imperative.md](imperative.md).
