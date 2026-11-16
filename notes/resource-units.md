# Resource units — CPU and memory

The units used in `resources.requests` and `resources.limits` ([chapter 03-07](03-scheduling/07-resource-requirements.md)), and in `kubectl top`, quotas and LimitRanges. Keep this open until the conversions are automatic.

---

## 1. CPU: whole CPUs and millicores

**1 CPU = 1000m.** The `m` means milli: one thousandth of a CPU, called a millicore or millicpu.

| Written as | Same as | Meaning |
|---|---|---|
| `2` | `2000m` | Two full CPUs |
| `1` | `1000m` | One full CPU |
| `0.5` | `500m` | Half a CPU |
| `0.25` | `250m` | A quarter |
| `0.1` | `100m` | A tenth |
| `0.001` | `1m` | The smallest amount allowed |

Count the digits: in millicores, **1 CPU has three zeros** (1000m), so 100m is 0.1 and 10m is 0.01. A CPU amount finer than `1m` is not allowed, which is why the docs recommend always writing CPU in millicores below 1.

What "1 CPU" is:
- One physical core, or one virtual CPU / hyperthread, depending on the node: one AWS vCPU, one GCP or Azure core.
- An **absolute** amount: `500m` is the same CPU time on a 2-core node as on a 64-core node.

How the kernel applies it (Linux cgroups):
- A **limit** of `500m` means the container gets at most 50 ms of CPU time in every 100 ms period. Once it has used that, it waits until the next period. This is **throttling**: the app slows down but keeps running.
- A **request** sets the container's share when containers compete for CPU. With no competition, a container may use more than its request, up to its limit.

---

## 2. Memory: two scales

Memory is counted in **bytes**. There are two sets of suffixes, and Kubernetes accepts both:

| Decimal (SI, powers of 10) | Bytes | Binary (IEC, powers of 2) | Bytes |
|---|---|---|---|
| `k` kilobyte | 1,000 | `Ki` kibibyte | 1,024 |
| `M` megabyte | 1,000,000 | `Mi` mebibyte | 1,048,576 |
| `G` gigabyte | 1,000,000,000 | `Gi` gibibyte | 1,073,741,824 |
| `T` terabyte | 1,000,000,000,000 | `Ti` tebibyte | 1,099,511,627,776 |

The binary unit is always the larger, and the gap grows with each step:

| Pair | Difference |
|---|---|
| `Ki` vs `k` | 2.4% |
| `Mi` vs `M` | 4.9% |
| `Gi` vs `G` | 7.4% |
| `Ti` vs `T` | 10% |

Operating systems and `kubectl top` count in powers of 2, while disk makers and network speeds use powers of 10. That is why a "500 GB" disk shows as about 465 GiB. Use the `i` units (`Mi`, `Gi`) for memory in Kubernetes. They match what the kernel and the tools report.

Useful conversions:

| Value | Equals |
|---|---|
| `1Gi` | `1024Mi` |
| `0.5Gi` | `512Mi` |
| `1.5Gi` | `1536Mi` (this is how `kubectl` writes it back) |
| `1G` | about `954Mi` |
| `500M` | about `477Mi` |
| `128Mi` | 134,217,728 bytes, about `134M` |

### 2.1 Case matters

| Written | Means | Notes |
|---|---|---|
| `1k` | 1,000 bytes | Lowercase `k` is the only valid decimal kilo |
| `1K` | **Invalid** | `kubectl` rejects it with `unable to parse quantity's suffix`. The course's slide uses `K` |
| `1Ki` | 1,024 bytes | |
| `400M` | 400 megabytes | |
| `400m` | **0.4 bytes** | Lowercase `m` is milli, even for memory. You almost certainly meant `400Mi` |
| `0.5` | 0.5 bytes, stored as `500m` | A bare decimal means bytes, not gigabytes |

---

## 3. Bits and bytes

1 byte = 8 bits. Memory and disk use bytes (capital **B**: MB, GiB). Network speeds use bits (lowercase **b**: Mbps, Gbps). A 1 Gbps link moves at most about 125 MB per second. Kubernetes resource quantities are always bytes.

---

## 4. Quoting in YAML

```yaml
resources:
  requests:
    cpu: 250m          # fine unquoted
    memory: 64Mi       # fine unquoted
  limits:
    cpu: "1"           # quote bare numbers so they stay strings; 1 also works
    memory: 128Mi
```

Kubernetes accepts both `cpu: 1` and `cpu: "1"`, and writes whole numbers back as strings (`"2"`).

## References

- [Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) — "Resource units in Kubernetes": CPU units, memory units, and the `400m` warning
- [Quantity](https://kubernetes.io/docs/reference/kubernetes-api/definitions/quantity-resource/) — the full suffix grammar
