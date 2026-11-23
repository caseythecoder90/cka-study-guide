# 03 — Commands and Arguments in Kubernetes

A container spec has two fields that override the image's defaults ([02](02-commands-and-arguments-docker.md)):

| Pod field | Overrides | Docker equivalent |
|---|---|---|
| `command` | the image's `ENTRYPOINT` | `docker run --entrypoint` |
| `args` | the image's `CMD` | arguments after the image name |

The name `command` is misleading: it replaces ENTRYPOINT, not CMD.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper          # ENTRYPOINT ["sleep"], CMD ["5"]
    command: ["sleep2.0"]          # replaces ENTRYPOINT
    args: ["10"]                   # replaces CMD   → runs: sleep2.0 10
```

---

## 1. What actually runs

With the image `ENTRYPOINT ["sleep"]` and `CMD ["5"]`:

| `command` | `args` | Runs | Why |
|---|---|---|---|
| — | — | `sleep 5` | image defaults |
| — | `["10"]` | `sleep 10` | args replace CMD only |
| `["sleep2.0"]` | — | `sleep2.0` | **setting `command` also drops the image's CMD** |
| `["sleep2.0"]` | `["10"]` | `sleep2.0 10` | both replaced |

The third row is the trap. If you set only `command`, the image's default arguments are gone too. Put everything you need in `command`, or add `args`.

## 2. Writing the values

Both fields are **lists of strings**. Each argument is a separate element.

```yaml
command: ["sleep", "3600"]         # flow list
args: ["10"]                       # numbers must be quoted: args: [10] is rejected
command:                           # block list, the same thing
- sleep
- "3600"
command: ["sh", "-c", "echo start; sleep 3600"]   # several commands, or pipes: run them through a shell
args: ["--port", "$(PORT)"]        # $(VAR) expands from the container's env
```

- **Unquoted numbers:** the API expects strings, so `args: [10]` fails with a "cannot unmarshal number" error.
- **Combined strings:** `["sleep 3600"]` looks for an executable named `sleep 3600`. The container fails to start (`StartError`, "executable file not found").

### 2.1 When you need `sh -c`

`command` is **not run through a shell**. The first element is executed directly and the rest are passed to it as arguments.

**Use a shell only when the command line needs shell features:**
- several commands (`;` or `&&`);
- pipes (`|`) or redirects (`>`, `>>`);
- loops;
- `$VAR` (without parentheses);
- wildcards such as `*.log`.

```yaml
command: ["sleep", "5000"]                                   # one program: no shell needed
command: ["sh", "-c", "while true; do date >> /data/out.txt; sleep 5; done"]   # loop + redirect: shell
```

**Always use `-c`, followed by the whole command line as ONE string.** Without `-c`, `sh` treats its first argument as a **script file to read**:
- `["/bin/sh", "sleep", "5000"]` makes `sh` try to open a file called `sleep`.
- It fails with `sh: 0: cannot open sleep: No such file`, and the Pod goes into CrashLoopBackOff.

Two related points:
- Kubernetes expands `$(VAR)` itself, so `["--port", "$(PORT)"]` needs no shell.
- A distroless image (such as kube-apiserver) has no shell, so `sh -c` cannot work there.

## 3. Imperative

```bash
k run sleeper --image=busybox $do -- sleep 3600                # after --: args
k run sleeper --image=busybox $do --command -- sleep 3600      # with --command: command
k run web --image=nginx $do -- --port=8080                     # args to the image's entrypoint
```

`kubectl run` without `--command` puts everything after `--` into **`args`**. For `busybox`, which has no ENTRYPOINT and `CMD ["sh"]`, both forms run the same thing. For an image with an ENTRYPOINT, they don't. Check the generated YAML before applying.

## 4. Changing it on a running Pod

`command` and `args` cannot be changed on an existing Pod. `kubectl edit` refuses to save and writes your edit to a temporary file:

```bash
k edit pod sleeper                  # error: ... saved to /tmp/kubectl-edit-xxxx.yaml
k replace --force -f /tmp/kubectl-edit-xxxx.yaml   # delete and recreate with the change
k get pod sleeper -o jsonpath='{.spec.containers[0].command} {.spec.containers[0].args}'
```

In a Deployment, edit the template instead (`spec.template.spec.containers[].command`). That starts a normal rollout ([01](01-rolling-updates-and-rollbacks.md)).

---

## Exam angle

- **"Create a Pod that runs `sleep 5000`":** use `k run x --image=busybox $do -- sleep 5000 > x.yaml`, then apply. Verify with `k get pod x -o jsonpath='{.spec.containers[0].args}'`.
- **"What command does Pod/image X run?":** read `command` and `args` in `k get pod -o yaml`. If they're empty, the image's defaults apply; `crictl inspecti` on the node shows them.
- **Change the command of a running Pod:** `k edit` fails and saves the edit to `/tmp`. Run `k replace --force -f` on that file.
- **Traps:** unquoted numbers; `"sleep 3600"` as one element; `["sh", "sleep", "5000"]` without `-c`; setting `command` and expecting the image's CMD to still apply; forgetting `--command` on an image with an ENTRYPOINT.
- **Recall the fields:** `kubectl explain pod.spec.containers.command`.

## References

- [Define a command and arguments for a container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/) — the Pod YAML, `$(VAR)` expansion, and "Run a command in a shell"
- [Pod API reference](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/) — the Container table describes `command` ("Entrypoint array. Not executed within a shell") and `args`
- [kubectl run](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_run/) — `--command` and the `--` separator, with examples

## Related notes

- CKAD: [Commands and arguments](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/02-commands-and-arguments.md) — the developer's view
