# 02 — Commands and Arguments in Docker

A container runs **one main process** and lives exactly as long as that process does. When the process exits, the container exits.

A container is not a VM. It has no kernel and no init system of its own. The `ubuntu` image contains Ubuntu's userland files, but nothing starts and stays running unless the image tells it to.

The image's `ENTRYPOINT` and `CMD` decide which process that is. The next chapter maps them to a Pod's `command` and `args`.

---

## 1. Why `docker run ubuntu` exits at once

The `ubuntu` image's default is `CMD ["bash"]`. `docker run` attaches no terminal and no input, so bash reads end-of-input, has nothing to do, and exits with status 0. `docker ps -a` shows the container as `Exited (0)`.

- `docker run -it ubuntu` keeps it alive, because bash now has a terminal.
- `docker run ubuntu sleep 5` replaces the default command, so the container runs for 5 seconds.

Images built to serve end with a long-running process instead: `CMD ["nginx", "-g", "daemon off;"]`, or `ENTRYPOINT ["/entrypoint.sh"]` plus `CMD ["mysqld"]`.

## 2. ENTRYPOINT versus CMD

| Dockerfile | `docker run img` | `docker run img 10` |
|---|---|---|
| `CMD ["sleep", "5"]` | `sleep 5` | `10` (the whole command is replaced, which fails) |
| `ENTRYPOINT ["sleep"]` | `sleep`, which fails with "missing operand" | `sleep 10` |
| `ENTRYPOINT ["sleep"]` + `CMD ["5"]` | `sleep 5` | `sleep 10` |
| any of the above, with `docker run --entrypoint sleep2.0 img 10` | — | `sleep2.0 10` |

The rules behind the table:
- **ENTRYPOINT** is the executable.
- **CMD** is its default arguments, or the whole default command when there is no ENTRYPOINT.
- Arguments on the command line **replace CMD** and never replace ENTRYPOINT.
- `--entrypoint` replaces the ENTRYPOINT and also drops the image's CMD.

Note that `docker run ubuntu-sleeper sleep 10` on a CMD-only image works only because the whole command is replaced, `sleep` included.

## 3. Exec form versus shell form

```dockerfile
CMD ["sleep", "5"]     # exec form (JSON array): runs sleep directly as PID 1
CMD sleep 5            # shell form: runs /bin/sh -c "sleep 5"; sh is PID 1
CMD ["sleep 5"]        # wrong: looks for an executable literally named "sleep 5"
```

- In exec form, **each argument is its own array element**, and the first element is the executable.
- Use exec form for both instructions when combining them. A shell-form ENTRYPOINT ignores CMD and any command-line arguments.
- In shell form, `sh` is PID 1 and does not forward `SIGTERM`. So the app may not shut down cleanly when Kubernetes stops the Pod; it gets killed after the grace period instead.

**Slide correction:** the last slide shows `sleep: missing operand` under `ENTRYPOINT ["sleep"]` with `CMD ["5"]`. That output was copied from the previous case. With a CMD default, `docker run ubuntu-sleeper` runs `sleep 5`.

## 4. Finding an image's defaults

```bash
docker image inspect ubuntu --format 'EP={{json .Config.Entrypoint}} CMD={{json .Config.Cmd}}'
crictl inspecti <image> | grep -A3 -E '"(Entrypoint|Cmd)"'    # on a Kubernetes node: no docker there
```

---

## Exam angle

- No task tests Dockerfiles directly. The Pod-level `command` and `args` ([next chapter](03-commands-and-arguments-kubernetes.md)) are what get tested, and they rely on these rules.
- **A Pod exits immediately or shows `Completed` or `CrashLoopBackOff`:** its main process ended. Check what the image runs (`crictl inspecti`) and whether the Pod's command keeps something running, such as `sleep 3600`.
- **Trap:** `["sleep 5"]` as a single string. Each argument must be a separate element.

## References

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/) — "CMD", "ENTRYPOINT", and "Understand how CMD and ENTRYPOINT interact" (the full combination table)
- [Define a command and arguments for a container](https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/) — how this maps to Pods (next chapter)

## Related notes

- CKAD: [Commands and arguments](https://github.com/caseythecoder90/ckad-exam-prep/blob/main/notes/02-configuration/02-commands-and-arguments.md) — the developer's view
